# Database Schema: Omie's Swift Logistics 🗄️

## Overview

PostgreSQL (Supabase) schema for managing users, orders, riders, payments, and support interactions.

---

## Core Tables

### 1. Users Table

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  phone_number VARCHAR(20) UNIQUE NOT NULL,
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  profile_picture_url TEXT,
  status VARCHAR(50) DEFAULT 'active',
  email_verified BOOLEAN DEFAULT FALSE,
  phone_verified BOOLEAN DEFAULT FALSE,
  customer_tier VARCHAR(50) DEFAULT 'regular',
  total_orders INT DEFAULT 0,
  average_rating DECIMAL(3, 2),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_status ON users(status);
```

### 2. Riders Table

```sql
CREATE TABLE riders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  phone_number VARCHAR(20) UNIQUE NOT NULL,
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  profile_picture_url TEXT,
  status VARCHAR(50) DEFAULT 'offline',
  vehicle_type VARCHAR(50),
  id_verified BOOLEAN DEFAULT FALSE,
  total_deliveries INT DEFAULT 0,
  average_rating DECIMAL(3, 2),
  acceptance_rate DECIMAL(5, 2),
  wallet_balance DECIMAL(15, 2) DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_riders_status ON riders(status);
```

### 3. Orders Table

```sql
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_number VARCHAR(50) UNIQUE,
  user_id UUID NOT NULL REFERENCES users(id),
  rider_id UUID REFERENCES riders(id),
  pickup_address TEXT NOT NULL,
  dropoff_address TEXT NOT NULL,
  status VARCHAR(50) DEFAULT 'pending',
  item_description TEXT,
  total_fare DECIMAL(15, 2),
  payment_status VARCHAR(50) DEFAULT 'pending',
  user_rating INT,
  user_comment TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  completed_at TIMESTAMP,
  cancelled_at TIMESTAMP,
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
```

### 4. Chat Sessions Table

```sql
CREATE TABLE chat_sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  order_id UUID REFERENCES orders(id),
  escalated_to_agent_id UUID,
  started_at TIMESTAMP DEFAULT NOW(),
  ended_at TIMESTAMP,
  sentiment_score DECIMAL(3, 2),
  resolution_type VARCHAR(50),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_chats_user ON chat_sessions(user_id);
```

### 5. Chat Messages Table

```sql
CREATE TABLE chat_messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id UUID NOT NULL REFERENCES chat_sessions(id),
  role VARCHAR(20),
  content TEXT NOT NULL,
  message_type VARCHAR(50),
  metadata JSONB,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_messages_session ON chat_messages(session_id);
```

### 6. Payments Table

```sql
CREATE TABLE payments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID NOT NULL REFERENCES orders(id),
  user_id UUID NOT NULL REFERENCES users(id),
  amount DECIMAL(15, 2),
  payment_method VARCHAR(50),
  status VARCHAR(50) DEFAULT 'pending',
  gateway_transaction_id VARCHAR(255),
  created_at TIMESTAMP DEFAULT NOW(),
  completed_at TIMESTAMP
);

CREATE INDEX idx_payments_order ON payments(order_id);
```

### 7. Support Tickets Table

```sql
CREATE TABLE support_tickets (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  ticket_number VARCHAR(50) UNIQUE,
  user_id UUID NOT NULL REFERENCES users(id),
  order_id UUID REFERENCES orders(id),
  category VARCHAR(100),
  subject VARCHAR(255),
  description TEXT,
  status VARCHAR(50) DEFAULT 'open',
  assigned_to_id UUID,
  created_at TIMESTAMP DEFAULT NOW(),
  resolved_at TIMESTAMP,
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_tickets_user ON support_tickets(user_id);
CREATE INDEX idx_tickets_status ON support_tickets(status);
```

---

## Entity Relationships

```
Users
  ├── Orders (1:N)
  ├── Chat Sessions (1:N)
  ├── Payments (1:N)
  └── Support Tickets (1:N)

Riders
  ├── Orders (1:N)
  └── Transactions (1:N)

Orders
  ├── Payments (1:N)
  ├── Chat Sessions (1:N)
  └── Support Tickets (1:N)

Chat Sessions
  ��── Chat Messages (1:N)
  └── Support Tickets (1:N)
```

---

## Sample Query Patterns

### Get User's Active Orders
```sql
SELECT * FROM orders
WHERE user_id = $1
  AND status NOT IN ('completed', 'cancelled')
ORDER BY created_at DESC;
```

### Get Chat with Messages
```sql
SELECT 
  cs.*,
  json_agg(cm.* ORDER BY cm.created_at) as messages
FROM chat_sessions cs
LEFT JOIN chat_messages cm ON cs.id = cm.session_id
WHERE cs.id = $1
GROUP BY cs.id;
```

---

**Last Updated**: June 2026 | **Version**: 1.0 | **PostgreSQL Version**: 14+