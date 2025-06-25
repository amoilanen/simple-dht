# DHT System Test Plan

## 1. Basic Node Operations

### 1.1 Single Node Setup
```bash
# Start a single node
cargo run -- 4000
```

### 1.2 Multi-Node Network Setup
```bash
# Start bootstrap node
cargo run -- 4000

# Start nodes connecting to bootstrap
cargo run -- 4001 127.0.0.1:4000
cargo run -- 4002 127.0.0.1:4000
cargo run -- 4003 127.0.0.1:4000
```

## 2. Data Storage and Retrieval Tests

### 2.1 Basic Store and Find
```bash
# Connect to node 1
nc 127.0.0.1 4001
{"StoreString": ["key1", "value1"]}

{"FindStringValue": "key1"}

# Connect to node 3 (should retrieve same value)
nc 127.0.0.1 4003
{"FindStringValue": "key1"}
```

### 2.2 Multiple Key-Value Pairs
```bash
# Connect to node 1
nc 127.0.0.1 4001
{"StoreString": ["key1", "value1"]}
{"StoreString": ["key2", "value2"]}
{"StoreString": ["key3", "value3"]}
{"FindStringValue": "key1"}
{"FindStringValue": "key2"}
{"FindStringValue": "key3"}

# Connect to node 2
nc 127.0.0.1 4002
{"FindStringValue": "key1"}
{"FindStringValue": "key2"}
{"FindStringValue": "key3"}
```

### 2.3 Large Value Storage
```bash
# Generate a 1MB random string
dd if=/dev/urandom bs=1M count=1 | base64 > large_value.txt

# Store and retrieve large value
nc 127.0.0.1 4001
{"StoreString": ["large_key", "$(cat large_value.txt)"]}
{"FindStringValue": "large_key"}
```

## 3. Network Resilience Tests

### 3.1 Node Failure Recovery
```bash
# Start 4 nodes
cargo run -- 4000
cargo run -- 4001 127.0.0.1:4000
cargo run -- 4002 127.0.0.1:4000
cargo run -- 4003 127.0.0.1:4000

# Store data
nc 127.0.0.1 4001
{"StoreString": ["test_key", "test_value"]}

# Kill node 4001
pkill -f "cargo run -- 4001"

# Try to retrieve from other nodes
nc 127.0.0.1 4002
{"FindStringValue": "test_key"}

nc 127.0.0.1 4003
{"FindStringValue": "test_key"}
```

### 3.2 Bootstrap Node Failure
```bash
# Start network
cargo run -- 4000
cargo run -- 4001 127.0.0.1:4000
cargo run -- 4002 127.0.0.1:4000

# Store data
nc 127.0.0.1 4001
{"StoreString": ["test_key", "test_value"]}

# Kill bootstrap node
pkill -f "cargo run -- 4000"

# Try to retrieve from remaining nodes
nc 127.0.0.1 4001
{"FindStringValue": "test_key"}

nc 127.0.0.1 4002
{"FindStringValue": "test_key"}
```

## 4. Concurrent Operations Tests

### 4.1 Multiple Concurrent Stores
```bash
# Start network
cargo run -- 4000
cargo run -- 4001 127.0.0.1:4000
cargo run -- 4002 127.0.0.1:4000

# In separate terminals, connect to different nodes and store values
# Terminal 1
nc 127.0.0.1 4001
{"StoreString": ["key1", "value1"]}
{"StoreString": ["key2", "value2"]}

# Terminal 2
nc 127.0.0.1 4002
{"StoreString": ["key3", "value3"]}
{"StoreString": ["key4", "value4"]}

# Verify all values are accessible from any node
nc 127.0.0.1 4001
{"FindStringValue": "key1"}
{"FindStringValue": "key2"}
{"FindStringValue": "key3"}
{"FindStringValue": "key4"}
```

### 4.2 Concurrent Find Operations
```bash
# Start network and store data
cargo run -- 4000
cargo run -- 4001 127.0.0.1:4000
cargo run -- 4002 127.0.0.1:4000

nc 127.0.0.1 4001
{"StoreString": ["test_key", "test_value"]}

# In separate terminals, perform concurrent finds
# Terminal 1
nc 127.0.0.1 4001
{"FindStringValue": "test_key"}

# Terminal 2
nc 127.0.0.1 4002
{"FindStringValue": "test_key"}
```

## 5. Edge Cases

### 5.1 Non-existent Keys
```bash
# Start network
cargo run -- 4000
cargo run -- 4001 127.0.0.1:4000

# Try to find non-existent key
nc 127.0.0.1 4001
{"FindStringValue": "nonexistent_key"}
```

### 5.2 Empty Values
```bash
# Start network
cargo run -- 4000
cargo run -- 4001 127.0.0.1:4000

# Store and retrieve empty value
nc 127.0.0.1 4001
{"StoreString": ["empty_key", ""]}
{"FindStringValue": "empty_key"}
```

### 5.3 Special Characters
```bash
# Start network
cargo run -- 4000
cargo run -- 4001 127.0.0.1:4000

# Store and retrieve values with special characters
nc 127.0.0.1 4001
{"StoreString": ["special_key", "!@#$%^&*()_+"]}
{"FindStringValue": "special_key"}
```

## 6. Performance Tests

### 6.1 Bulk Operations
```bash
# Start network
cargo run -- 4000
cargo run -- 4001 127.0.0.1:4000

# Store 1000 key-value pairs
for i in {1..1000}; do
    echo "{\"StoreString\": [\"key$i\", \"value$i\"]}" | nc 127.0.0.1 4001
done

# Retrieve all values
for i in {1..1000}; do
    echo "{\"FindStringValue\": \"key$i\"}" | nc 127.0.0.1 4001
done
```

### 6.2 Network Load Test
```bash
# Start 5 nodes
cargo run -- 4000
cargo run -- 4001 127.0.0.1:4000
cargo run -- 4002 127.0.0.1:4000
cargo run -- 4003 127.0.0.1:4000
cargo run -- 4004 127.0.0.1:4000

# In separate terminals, perform operations on different nodes
# Terminal 1
nc 127.0.0.1 4001
{"StoreString": ["key1", "value1"]}
{"FindStringValue": "key1"}

# Terminal 2
nc 127.0.0.1 4002
{"StoreString": ["key2", "value2"]}
{"FindStringValue": "key2"}

# Terminal 3
nc 127.0.0.1 4003
{"StoreString": ["key3", "value3"]}
{"FindStringValue": "key3"}

# Terminal 4
nc 127.0.0.1 4004
{"StoreString": ["key4", "value4"]}
{"FindStringValue": "key4"}
```

## 7. Security Tests

### 7.1 Invalid Commands
```bash
# Start network
cargo run -- 4000
cargo run -- 4001 127.0.0.1:4000

# Try invalid commands
nc 127.0.0.1 4001
INVALID_COMMAND
{"InvalidCommand": []}
{"StoreString": []}
{"FindStringValue": []}
{"StoreString": ["key"]}
{"FindStringValue": "key", "extra": "argument"}
```

### 7.2 Malformed Input
```bash
# Start network
cargo run -- 4000
cargo run -- 4001 127.0.0.1:4000

# Try malformed input
nc 127.0.0.1 4001
{"StoreString": ["key with spaces", "value with spaces"]}
{"StoreString": ["key\"with\"quotes", "value"]}
{"StoreString": ["key", "value with spaces"]}
```

## 8. Smoke Tests

### 8.1 Basic cluster

```bash
cargo run -- 4000
cargo run -- 4001 127.0.0.1:4000
cargo run -- 4002 127.0.0.1:4000
cargo run -- 4003 127.0.0.1:4000

nc 127.0.0.1 4001
{"StoreString": ["hello", "world"]}
{"FindStringValue": "hello"}

nc 127.0.0.1 4003
{"FindStringValue": "hello"}
```