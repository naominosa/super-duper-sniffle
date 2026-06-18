# AI & Chatbot Strategy: Omie's Swift Logistics 🤖

## Overview

A **human-centered AI chatbot system** designed to provide 24/7 support while maintaining authentic, empathetic interactions. The AI prioritizes understanding user context and seamlessly escalating to human agents when needed.

---

## 1. Core Philosophy

### Not Another Generic Bot

Traditional chatbots feel robotic and frustrating. Omie's AI prioritizes:

✅ **Contextual Understanding** - Knows order history, rider info, location context  
✅ **Empathy First** - Acknowledges frustration before solving  
✅ **Natural Language** - Uses Nigerian English, colloquial phrases  
✅ **Fast Escalation** - Gets humans involved quickly when needed  
✅ **Transparency** - Admits when it can't help, never deceives  

### Guiding Principle
> "The best AI is one that doesn't feel like AI. Use technology to feel more human."

---

## 2. Architecture

### System Diagram

```
User Message
    │
    ▼
┌─────────────────────────────┐
│ Context Aggregator          │
│ • User profile              │
│ • Order history             │
│ • Rider data                │
│ • Location context          │
└──────────────┬──────────────┘
               │
    ┌──────────▼──────────┐
    │ Sentiment Analysis   │
    │ (threshold-based)    │
    └──────────┬───────────┘
               │
    ┌──────────▼────────────────┐
    │ Route Decision Engine      │
    ├──────────────────────────┤
    │ • Simple FAQ? → Direct   │
    │ • Complex? → AI Gen      │
    │ • Angry? → Escalate      │
    │ • Long wait? → Escalate  │
    └──────────┬────────────────┘
               │
        ┌──────┴──────┐
        │             │
        ▼             ▼
    ┌───────┐    ┌──────────────┐
    │ FAQ   │    │ Claude 3.5   │
    │ Rules │    │ Sonnet API   │
    └───────┘    └──────────────┘
        │             │
        └──────┬──────┘
               │
               ▼
    ┌─────────────────────────┐
    │ Response Generator      │
    │ • Add human touch       │
    │ • Format for UI         │
    │ • Add quick actions     │
    └──────────┬──────────────┘
               │
               ▼
        Return to User
```

---

## 3. Implementation Roadmap

### Phase 1: Foundation (Weeks 1-2)

#### 3.1.1 Database Schema for Chat

```sql
-- Chat Sessions Table
CREATE TABLE chat_sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  order_id UUID REFERENCES orders(id),
  started_at TIMESTAMP DEFAULT NOW(),
  ended_at TIMESTAMP,
  escalated_to_agent_id UUID REFERENCES support_agents(id),
  sentiment_score FLOAT,
  resolution_type VARCHAR(50), -- 'ai_resolved', 'agent_resolved', 'unresolved'
  created_at TIMESTAMP DEFAULT NOW()
);

-- Chat Messages Table
CREATE TABLE chat_messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id UUID NOT NULL REFERENCES chat_sessions(id),
  role VARCHAR(20), -- 'user', 'assistant', 'agent'
  content TEXT NOT NULL,
  message_type VARCHAR(50), -- 'text', 'suggestion', 'action_card'
  metadata JSONB, -- Extra data (sentiment, action buttons, etc.)
  created_at TIMESTAMP DEFAULT NOW()
);

-- Escalation Queue
CREATE TABLE escalation_queue (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  chat_session_id UUID NOT NULL REFERENCES chat_sessions(id),
  reason VARCHAR(100), -- 'sentiment_negative', 'timeout', 'user_requested'
  priority VARCHAR(20), -- 'low', 'normal', 'high', 'vip'
  assigned_agent_id UUID REFERENCES support_agents(id),
  wait_time_seconds INT,
  created_at TIMESTAMP DEFAULT NOW(),
  assigned_at TIMESTAMP,
  resolved_at TIMESTAMP
);
```

#### 3.1.2 Context Aggregation Service

```typescript
// backend/src/services/chatContextService.ts

interface ChatContext {
  user: UserProfile;
  currentOrder: OrderDetails | null;
  recentOrders: Order[];
  riderInfo: RiderProfile | null;
  locationContext: LocationData;
  customerTier: 'vip' | 'regular' | 'new';
  sentimentHistory: SentimentScore[];
  previousIssues: SupportTicket[];
}

export async function aggregateChatContext(
  userId: string,
  orderId?: string
): Promise<ChatContext> {
  const user = await db.users.findById(userId);
  const currentOrder = orderId ? await db.orders.findById(orderId) : null;
  const recentOrders = await db.orders.findRecent(userId, 5);
  
  const riderInfo = currentOrder 
    ? await db.riders.findById(currentOrder.rider_id)
    : null;

  // Calculate customer tier based on order history
  const customerTier = calculateTier(user, recentOrders);
  
  // Fetch sentiment from previous chats
  const sentimentHistory = await db.chatSessions
    .findBySentiment(userId)
    .orderBy('created_at', 'DESC')
    .limit(10);

  // Identify recurring issues
  const previousIssues = await db.supportTickets
    .findByUser(userId)
    .orderBy('created_at', 'DESC')
    .limit(5);

  return {
    user,
    currentOrder,
    recentOrders,
    riderInfo,
    locationContext: extractLocationContext(user, currentOrder),
    customerTier,
    sentimentHistory,
    previousIssues,
  };
}
```

#### 3.1.3 Sentiment Analysis Integration

```typescript
// backend/src/services/sentimentService.ts

import { Anthropic } from '@anthropic-ai/sdk';

const anthropic = new Anthropic();

interface SentimentResult {
  score: number; // -1 (very negative) to 1 (very positive)
  emotion: 'angry' | 'frustrated' | 'neutral' | 'satisfied' | 'happy';
  confidence: number;
  reasoning: string;
}

export async function analyzeSentiment(
  message: string,
  context: ChatContext
): Promise<SentimentResult> {
  const response = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 200,
    messages: [
      {
        role: 'user',
        content: `Analyze the sentiment of this customer message in context of their order status.

Customer Message: "${message}"

Context:
- Order Status: ${context.currentOrder?.status}
- Wait Time: ${calculateWaitTime(context.currentOrder)} minutes
- Customer Tier: ${context.customerTier}
- Previous Issue: ${context.previousIssues[0]?.description || 'None'}

Return JSON with: score (-1 to 1), emotion, confidence (0-1), reasoning`,
      },
    ],
  });

  const text = response.content[0].type === 'text' ? response.content[0].text : '';
  const result = JSON.parse(text);
  
  return result;
}

export function shouldEscalate(sentiment: SentimentResult, context: ChatContext): boolean {
  // Rule-based escalation logic
  const rules = [
    sentiment.score < -0.6, // Very negative sentiment
    sentiment.emotion === 'angry', // Angry emotion detected
    context.sentimentHistory.filter(s => s.score < -0.3).length > 2, // Recurring issues
    context.customerTier === 'vip' && sentiment.score < 0, // VIP with any negativity
  ];

  return rules.some(rule => rule === true);
}
```

---

### Phase 2: Claude Integration (Weeks 2-3)

#### 3.2.1 System Prompt Construction

```typescript
// backend/src/services/aiChatbotService.ts

function buildSystemPrompt(context: ChatContext): string {
  return `You are Omie, the friendly support assistant for Omie's Swift Logistics.
  
Your personality:
- Warm, human, and empathetic
- Uses Nigerian English and colloquialisms where appropriate
- Funny but professional (jokes are okay if relevant)
- Always honest - admit when you don't know something
- Prioritize customer satisfaction over corporate-speak

Current Customer Context:
- Name: ${context.user.first_name}
- Account Status: ${context.user.status} customer (${context.recentOrders.length} total orders)
- Current Order: ${context.currentOrder ? `#${context.currentOrder.id} - ${context.currentOrder.status}` : 'No active order'}
- Loyalty Tier: ${context.customerTier === 'vip' ? '⭐ VIP' : 'Regular'}

Current Situation:
${buildSituationContext(context)}

Your Goals (in order):
1. Understand the customer's problem/question
2. Provide immediate help if possible
3. If not possible, escalate to a human agent
4. Always end with clear next steps

Response Guidelines:
- Keep responses to 2-3 sentences max (chat format)
- Use emojis sparingly (1-2 max per message)
- Always offer an action: solve it, escalate, or reschedule
- If customer is upset, acknowledge their feeling FIRST
- Include relevant quick-reply buttons when applicable

Special Instructions:
- Never make up information about orders or riders
- If order status is delayed, show empathy AND actionable solutions
- For payment issues, offer multiple payment methods
- Always provide a "Talk to a human" option

Examples of Good Responses:
- "I totally get it - waiting sucks! Your rider Chukwu is stuck in traffic on Lekki. 
  Should we reschedule for a better time, or would you like to chat with one of our team?"
  
- "Ah, this is the second time your card's been declined. Let's fix this quickly. 
  Try USSD (Dial *402*8#) or I can connect you with Tunde from our team?"
  
- "Sorry about that delay! As compensation, we'll credit ₦300 to your wallet. 
  Can we book another rider for you now?"

If customer gets angry or frustrated:
1. Do NOT get defensive
2. Acknowledge their emotion: "I hear your frustration, and you're right to be upset"
3. Explain what happened (briefly)
4. Offer solution PLUS escalation option
5. Get a human involved if sentiment score < -0.7`;
}

function buildSituationContext(context: ChatContext): string {
  let situation = '';
  
  if (context.currentOrder) {
    const waitTime = calculateWaitTime(context.currentOrder);
    situation += `\nOrder Status: ${context.currentOrder.status}
    - From: ${context.currentOrder.pickup_location}
    - To: ${context.currentOrder.dropoff_location}
    - Rider: ${context.riderInfo?.name || 'Unassigned'}
    - Estimated arrival: ${context.currentOrder.eta}
    - Wait time so far: ${waitTime} mins`;
  } else {
    situation += '\nNo active order at the moment';
  }

  if (context.previousIssues.length > 0) {
    situation += `\n\nRecent Issues:
    ${context.previousIssues.map(issue => `- ${issue.description}`).join('\n')}`;
  }

  return situation;
}
```

#### 3.2.2 API Call Handler

```typescript
// backend/src/services/aiChatbotService.ts

export async function generateAIResponse(
  userMessage: string,
  context: ChatContext,
  conversationHistory: ChatMessage[]
): Promise<{
  response: string;
  shouldEscalate: boolean;
  suggestedActions: ActionButton[];
}> {
  const sentiment = await analyzeSentiment(userMessage, context);

  // Check if escalation is needed
  const escalate = shouldEscalate(sentiment, context);

  if (escalate) {
    return {
      response: generateEscalationMessage(context),
      shouldEscalate: true,
      suggestedActions: [
        { label: 'Talk to a Human', action: 'escalate' },
        { label: 'Keep Chatting', action: 'continue' }
      ]
    };
  }

  // Build Claude prompt with conversation history
  const messages = [
    ...conversationHistory.map(msg => ({
      role: msg.role as 'user' | 'assistant',
      content: msg.content
    })),
    {
      role: 'user' as const,
      content: userMessage
    }
  ];

  const response = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 500,
    system: buildSystemPrompt(context),
    messages,
  });

  const assistantMessage = response.content[0].type === 'text' 
    ? response.content[0].text 
    : '';

  // Extract suggested actions from response
  const suggestedActions = extractSuggestedActions(
    userMessage,
    assistantMessage,
    context
  );

  return {
    response: assistantMessage,
    shouldEscalate: false,
    suggestedActions
  };
}

function generateEscalationMessage(context: ChatContext): string {
  const queue = context.customerTier === 'vip' 
    ? 'priority queue' 
    : 'support queue';

  return `I totally understand your situation. Let me connect you with someone from our team who can help right away. 
    You're in our ${queue} and should hear back in just 2-3 minutes. They'll have all your details ready! 🎯`;
}

function extractSuggestedActions(
  userMessage: string,
  aiResponse: string,
  context: ChatContext
): ActionButton[] {
  const actions: ActionButton[] = [];

  // Detect if user mentioned delay/cancellation
  if (userMessage.includes('late') || userMessage.includes('cancel')) {
    actions.push({
      label: 'Reschedule Order',
      action: 'reschedule'
    });
  }

  // Detect payment issues
  if (userMessage.includes('payment') || userMessage.includes('card')) {
    actions.push({
      label: 'Change Payment Method',
      action: 'payment'
    });
  }

  // Always offer escalation
  actions.push({
    label: 'Talk to a Human',
    action: 'escalate'
  });

  return actions;
}
```

---

### Phase 3: Escalation System (Week 3)

#### 3.3.1 Escalation Rules Engine

```typescript
// backend/src/services/escalationService.ts

interface EscalationRule {
  name: string;
  condition: (context: ChatContext, sentiment: SentimentResult) => boolean;
  priority: 'low' | 'normal' | 'high' | 'vip';
  timeout: number; // minutes before auto-escalate
  reason: string;
}

const escalationRules: EscalationRule[] = [
  {
    name: 'Angry Customer',
    condition: (ctx, sentiment) => sentiment.score < -0.7,
    priority: 'high',
    timeout: 2,
    reason: 'sentiment_very_negative',
  },
  {
    name: 'Order Significantly Delayed',
    condition: (ctx) => {
      if (!ctx.currentOrder) return false;
      const delayMinutes = calculateWaitTime(ctx.currentOrder);
      return delayMinutes > 15;
    },
    priority: 'normal',
    timeout: 5,
    reason: 'order_delayed',
  },
  {
    name: 'Recurring Payment Issues',
    condition: (ctx) => {
      const paymentIssues = ctx.previousIssues.filter(
        issue => issue.description.includes('payment')
      );
      return paymentIssues.length >= 2;
    },
    priority: 'normal',
    timeout: 3,
    reason: 'recurring_payment_issue',
  },
  {
    name: 'VIP Customer with Concern',
    condition: (ctx, sentiment) => {
      return ctx.customerTier === 'vip' && sentiment.score < 0;
    },
    priority: 'vip',
    timeout: 1,
    reason: 'vip_concern',
  },
  {
    name: 'Multiple Failed AI Attempts',
    condition: (ctx) => {
      // If AI fails to resolve after 3 turns, escalate
      return false; // Handled by turn counter separately
    },
    priority: 'normal',
    timeout: 4,
    reason: 'ai_max_attempts',
  }
];

export async function checkEscalationRules(
  context: ChatContext,
  sentiment: SentimentResult
): Promise<EscalationRule | null> {
  for (const rule of escalationRules) {
    if (rule.condition(context, sentiment)) {
      return rule;
    }
  }
  return null;
}

export async function escalateChatSession(
  sessionId: string,
  rule: EscalationRule,
  context: ChatContext
): Promise<void> {
  // Determine priority queue
  const priority = rule.priority === 'vip' ? 'vip' : 'normal';

  // Find available agent
  const agent = await findAvailableAgent(priority);

  // Create escalation record
  await db.escalationQueue.create({
    chat_session_id: sessionId,
    reason: rule.reason,
    priority: rule.priority,
    assigned_agent_id: agent?.id,
    created_at: new Date(),
  });

  // Notify agent
  await notifyAgent(agent, {
    sessionId,
    customerId: context.user.id,
    reason: rule.reason,
    context,
  });

  // Send confirmation to customer
  await sendCustomerNotification(context.user.id, {
    type: 'agent_assigned',
    message: `${agent?.name || 'A support agent'} is taking over your chat now. They'll solve this!`,
  });
}
```

#### 3.3.2 Support Agent Dashboard

```typescript
// backend/src/routes/agentDashboard.ts

interface AgentDashboardData {
  queue: EscalatedChat[];
  activeChats: ChatSession[];
  metrics: {
    avgResolutionTime: number;
    satisfactionScore: number;
    handledToday: number;
  };
}

app.get('/api/agent-dashboard', authenticateAgent, async (req, res) => {
  const agentId = req.user.id;

  const escalationQueue = await db.escalationQueue
    .findUnassigned()
    .orderBy('priority')
    .orderBy('created_at');

  const activeChats = await db.chatSessions
    .findByAgent(agentId)
    .where('ended_at', 'IS NULL');

  const metrics = await getAgentMetrics(agentId);

  res.json({
    queue: escalationQueue,
    activeChats,
    metrics,
  });
});

// Real-time updates via WebSocket
io.on('connection', (socket) => {
  if (socket.user.role === 'support_agent') {
    socket.on('chat:message', async (data) => {
      const { sessionId, message } = data;

      // Save message to DB
      await db.chatMessages.create({
        session_id: sessionId,
        role: 'agent',
        content: message,
        created_at: new Date(),
      });

      // Emit to customer
      io.to(`session_${sessionId}`).emit('chat:message', {
        role: 'agent',
        content: message,
        timestamp: new Date(),
      });
    });

    socket.on('chat:resolve', async (sessionId) => {
      await db.chatSessions.update(sessionId, {
        ended_at: new Date(),
        resolution_type: 'agent_resolved',
      });

      io.to(`session_${sessionId}`).emit('chat:resolved');
    });
  }
});
```

---

### Phase 4: Proactive Support (Week 4)

#### 3.4.1 Predictive Triggers

```typescript
// backend/src/jobs/proactiveSupport.ts

import { CronJob } from 'cron';

// Run every minute
const proactiveSupportJob = new CronJob('* * * * *', async () => {
  // Find all active orders
  const activeOrders = await db.orders
    .where('status', 'IN', ['pending', 'accepted', 'in_transit']);

  for (const order of activeOrders) {
    await checkOrderForProactiveAction(order);
  }
});

async function checkOrderForProactiveAction(order: Order) {
  const delayMinutes = calculateWaitTime(order);
  const user = await db.users.findById(order.user_id);

  // Trigger 1: Order is 5+ minutes late
  if (delayMinutes >= 5 && !order.proactive_message_sent) {
    await sendProactiveMessage(user, {
      type: 'order_delay',
      delayMinutes,
      suggestion: 'reschedule',
    });

    await db.orders.update(order.id, {
      proactive_message_sent: true
    });
  }

  // Trigger 2: No rider assigned after 2 minutes
  if (!order.rider_id && delayMinutes >= 2) {
    await sendProactiveMessage(user, {
      type: 'finding_rider',
      delayMinutes,
      suggestion: 'increase_fare_or_wait',
    });
  }

  // Trigger 3: Payment pending for 10+ minutes
  if (order.status === 'pending_payment' && delayMinutes >= 10) {
    await sendProactiveMessage(user, {
      type: 'payment_pending',
      suggestion: 'retry_or_different_method',
    });
  }
}

async function sendProactiveMessage(
  user: User,
  trigger: ProactiveTrigger
): Promise<void> {
  let message = '';

  switch (trigger.type) {
    case 'order_delay':
      message = `⏱️ Your rider is running ${trigger.delayMinutes} mins late due to traffic.
      Want to reschedule or adjust the fare? 🤔`;
      break;

    case 'finding_rider':
      message = `🔍 We're still finding a rider for you. 
      Want to increase the fare a bit to attract faster riders? 📈`;
      break;

    case 'payment_pending':
      message = `💳 Your payment is still pending. Try USSD or a different card?
      Need help? Chat with us! 💬`;
      break;
  }

  // Send via chat, push notification, and SMS
  await db.chatMessages.create({
    session_id: null, // Proactive system message
    role: 'assistant',
    content: message,
    message_type: 'proactive_alert',
  });

  await sendPushNotification(user, { title: 'Omie Update', body: message });
  await sendSMS(user.phone, message);
}
```

#### 3.4.2 Smart Suggestions

```typescript
// backend/src/services/suggestionEngine.ts

export function generateSuggestedActions(
  context: ChatContext,
  userMessage: string,
  aiResponse: string
): ActionButton[] {
  const actions: ActionButton[] = [];

  // Analyze user message intent
  const intent = detectIntent(userMessage);

  // Context-aware suggestions
  if (context.currentOrder?.status === 'pending') {
    if (intent === 'delay_inquiry') {
      actions.push({
        label: 'Track Your Order',
        action: 'open_tracking'
      });
    }
  }

  if (context.currentOrder?.status === 'accepted') {
    actions.push({
      label: 'Call Your Rider',
      action: 'call_rider'
    });
    actions.push({
      label: 'View Live Location',
      action: 'open_map'
    });
  }

  // Payment-related suggestions
  if (intent === 'payment' || intent === 'booking') {
    if (context.user.saved_cards.length === 0) {
      actions.push({
        label: 'Add Payment Method',
        action: 'add_payment'
      });
    }
  }

  // Always include fallback
  actions.push({
    label: 'Talk to Support',
    action: 'escalate'
  });

  return actions;
}

function detectIntent(message: string): string {
  const keywords = {
    delay_inquiry: ['late', 'where', 'when', 'delayed', 'how long'],
    payment: ['payment', 'card', 'charge', 'money', 'charge', 'paid'],
    booking: ['book', 'rider', 'available', 'cost', 'price'],
    cancellation: ['cancel', 'stop', 'don\'t want', 'change mind'],
    complaint: ['angry', 'frustrated', 'worst', 'terrible', 'bad'],
  };

  for (const [intent, words] of Object.entries(keywords)) {
    if (words.some(word => message.toLowerCase().includes(word))) {
      return intent;
    }
  }

  return 'general';
}
```

---

## 4. Human Touch Elements

### 4.1 Agent Personality Configuration

```typescript
// Each support agent has a personality profile
interface AgentPersonality {
  name: string;
  emoji: string;
  style: 'formal' | 'casual' | 'funny';
  languages: string[];
  strengths: string[];
  bio: string;
}

const agents: AgentPersonality[] = [
  {
    name: 'Tunde',
    emoji: '👨‍💼',
    style: 'casual',
    languages: ['en', 'yoruba'],
    strengths: ['payment_issues', 'technical_support'],
    bio: 'Tunde has been helping customers for 3 years. He\'s got answers!',
  },
  {
    name: 'Chioma',
    emoji: '👩‍💼',
    style: 'funny',
    languages: ['en', 'igbo'],
    strengths: ['rider_issues', 'escalations'],
    bio: 'Chioma is our calm problem-solver. She\'ll make you laugh while fixing things.',
  },
];

// Handoff includes agent intro
const handoffMessage = `✨ I'm connecting you with Chioma from our team.
She's an expert with rider issues and will solve this fast.
Chioma, meet ${user.first_name}! 👋`;
```

### 4.2 Empathy Protocols

```typescript
// Emotion-aware responses

const empathyResponses = {
  angry: {
    opening: "I completely understand your frustration, and you're absolutely right to be upset.",
    action: "Here's exactly what I'm going to do about it:",
    closing: "This shouldn't have happened, and we'll make sure it doesn't again."
  },
  frustrated: {
    opening: "I hear you - this is frustrating!",
    action: "Let's fix this right now:",
    closing: "You shouldn't have to deal with this."
  },
  confused: {
    opening: "No worries, let me break this down clearly.",
    action: "Here's what's happening and what we can do:",
    closing: "Does that make sense? Ask if anything's still unclear."
  },
  concerned: {
    opening: "I understand your concern.",
    action: "Here's what I can do to reassure you:",
    closing: "You're in safe hands."
  }
};

// Usage
function buildEmpathyResponse(emotion: string, context: ChatContext): string {
  const template = empathyResponses[emotion];
  if (!template) return '';

  return `${template.opening}

${template.action}
- [Solution 1]
- [Solution 2]

${template.closing}`;
}
```

### 4.3 Contextual Personality

```typescript
// AI adapts tone based on user profile

function adaptTone(context: ChatContext): string {
  if (context.customerTier === 'vip') {
    return `VIP_PRIORITY`; // Ultra-responsive, personalized
  }

  if (context.user.status === 'new') {
    return `WELCOMING`; // Extra helpful, explain everything
  }

  if (context.previousIssues.length >= 3) {
    return `EMPATHETIC`; // Acknowledge recurring problems
  }

  if (context.user.last_order_rating >= 4.8) {
    return `FRIENDLY_CASUAL`; // Relaxed, familiar tone
  }

  return `PROFESSIONAL`; // Standard friendly
}
```

---

## 5. Metrics & Monitoring

### 5.1 Success Metrics

```typescript
interface ChatMetrics {
  resolution_rate: number; // % of chats resolved by AI
  escalation_rate: number; // % escalated to humans
  avg_resolution_time: number; // minutes
  customer_satisfaction: number; // 1-5
  first_contact_resolution: number; // % resolved without escalation
  cost_per_resolution: number; // ₦
}

// Dashboard queries
async function getChatMetrics(period: 'daily' | 'weekly' | 'monthly') {
  const chats = await db.chatSessions
    .where('created_at', '>', getPeriodStart(period));

  const aiResolved = chats.filter(c => c.resolution_type === 'ai_resolved').length;
  const escalated = chats.filter(c => c.escalated_to_agent_id).length;

  return {
    resolution_rate: aiResolved / chats.length,
    escalation_rate: escalated / chats.length,
    avg_resolution_time: calculateAverage(chats.map(c => c.resolution_time)),
    customer_satisfaction: await calculateSatisfaction(chats),
    first_contact_resolution: (aiResolved / chats.length) * 100,
    cost_per_resolution: calculateCosts(chats) / aiResolved,
  };
}
```

### 5.2 Monitoring Dashboard

```
╔═══════════════════════════════════════════════╗
║  OMIE CHATBOT ANALYTICS                       ║
╠═══════════════════════════════════════════════╣
║                                               ║
║  Today's Performance:                         ║
║  • Messages: 2,341                           ║
║  • AI Resolution Rate: 78%                    ║
║  • Avg Response Time: 2.3s                    ║
║  • Escalation Rate: 15%                       ║
║  • Customer Satisfaction: 4.6/5 ⭐            ║
║                                               ║
║  Active Chats: 24                            ║
║  Agents On Duty: 8                           ║
║  Queue Wait Time: 2 mins                      ║
║                                               ║
║  ⚠️ Alerts:                                    ║
║  • Escalation queue building (8 pending)     ║
║  • 2 negative sentiment alerts                ║
║                                               ║
╚═══════════════════════════════════════════════╝
```

---

## 6. Continuous Improvement

### 6.1 Feedback Loop

```typescript
// After each chat, gather feedback

interface ChatFeedback {
  sessionId: string;
  userId: string;
  rating: 1 | 2 | 3 | 4 | 5;
  wasHelpful: boolean;
  suggestedAnswer: string; // Optional correction
  emotion: string;
  timestamp: Date;
}

// Use feedback to fine-tune AI responses
async function trainFromFeedback() {
  const poorChats = await db.chatFeedback
    .where('rating', '<', 3)
    .orderBy('created_at', 'DESC')
    .limit(100);

  // Analyze patterns in poor responses
  // Update system prompts accordingly
  // Log for team review
}
```

### 6.2 A/B Testing Response Styles

```typescript
// Test different empathy levels, response lengths, etc.

async function runAbTest() {
  const variants = [
    {
      id: 'variant_a',
      systemPrompt: buildSystemPrompt({ empathyLevel: 'high' }),
      sampleSize: 1000,
    },
    {
      id: 'variant_b',
      systemPrompt: buildSystemPrompt({ empathyLevel: 'standard' }),
      sampleSize: 1000,
    },
  ];

  // Randomize which variant each user gets
  // Measure satisfaction, resolution rate, escalation rate
  // Deploy winning variant after 7 days
}
```

---

## 7. Failure Scenarios & Fallbacks

### 7.1 API Failures

```typescript
// If Claude API is down, fall back to rule-based FAQ

const faqFallback = {
  'where is my rider': 'Open the map in the app to track your rider in real-time!',
  'payment failed': 'Try another card or use USSD. Still having issues? Chat with us!',
  'cancel order': 'You can cancel within 2 minutes free. After that, there\'s a 20% fee.',
  // ... more FAQs
};

async function generateResponse(message: string, context: ChatContext) {
  try {
    return await aiChatbot.generateResponse(message, context);
  } catch (error) {
    logger.error('Claude API error', error);
    
    // Fall back to FAQ matching
    const matches = findFaqMatches(message);
    if (matches.length > 0) {
      return matches[0];
    }

    // Last resort: escalate to human
    return {
      response: 'Let me connect you with our team right now. They\'ll help immediately!',
      shouldEscalate: true,
    };
  }
}
```

---

## 8. Privacy & Data Security

```typescript
// Ensure sensitive data is protected

const sensitivePatterns = [
  /\d{10,}/g, // Phone numbers
  /[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}/gi, // Emails
  /₦\s?\d+/g, // Prices
];

function sanitizeChatMessage(message: string): string {
  let sanitized = message;
  sensitivePatterns.forEach(pattern => {
    sanitized = sanitized.replace(pattern, '[REDACTED]');
  });
  return sanitized;
}

// Store encrypted chat history
async function saveChatMessage(message: ChatMessage) {
  const encrypted = encryptMessage(message.content);
  await db.chatMessages.create({
    ...message,
    content: encrypted,
  });
}
```

---

## 9. Deployment Checklist

- [ ] Claude API key configured in production
- [ ] Redis connection tested for session caching
- [ ] Sentiment analysis model trained on Nigerian context
- [ ] Support agent dashboard tested with live chats
- [ ] Escalation queue functional with agent notifications
- [ ] SMS/Push notifications configured
- [ ] Chat history encrypted
- [ ] Monitoring dashboards set up
- [ ] Load testing completed (1000+ concurrent chats)
- [ ] Backup LLM provider configured (fallback ready)
- [ ] Customer privacy policy updated
- [ ] Support team trained on new system

---

## 10. Success Criteria (Week 4 Review)

✅ **AI handles 75%+ of support queries** without escalation  
✅ **Average response time < 2 seconds**  
✅ **Customer satisfaction rating ≥ 4.5/5**  
✅ **Escalation to human within 2 minutes for 95% of cases**  
✅ **Cost per resolution < ₦50**  
✅ **Zero false escalations (AI knows when to give up)**  
✅ **Support team morale: agents appreciate AI as tool, not replacement**  

---

**Last Updated**: June 2026 | **Version**: 1.0 | **Status**: Ready for Implementation
