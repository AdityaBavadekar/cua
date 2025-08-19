# @trycua/agent

TypeScript SDK for CUA agent interaction. Connect to CUA agent proxies via HTTP/HTTPS or peer-to-peer (WebRTC) connections.

## Installation

```bash
npm install @trycua/agent
# or
pnpm add @trycua/agent
# or
yarn add @trycua/agent
```

## Usage

### Basic Usage

```typescript
import AgentClient from "@trycua/agent";

// Connect to HTTP server
const client = new AgentClient("https://localhost:8000");

// Connect to peer
const peerClient = new AgentClient("peer://my-agent-proxy");

// Send a simple text request
const response = await client.responses.create({
  model: "anthropic/claude-3-5-sonnet-20241022",
  input: "Write a one-sentence bedtime story about a unicorn."
});

console.log(response.result);
```

### Multi-modal Requests

```typescript
const response = await client.responses.create({
  model: "anthropic/claude-3-5-sonnet-20241022",
  input: [
    {
      role: "user",
      content: [
        { type: "input_text", text: "What is in this image?" },
        { 
          type: "input_image", 
          image_url: "https://example.com/image.jpg" 
        }
      ]
    }
  ]
});
```

### Advanced Configuration

```typescript
const client = new AgentClient("https://localhost:8000", {
  timeout: 60000, // 60 second timeout
  retries: 5      // 5 retry attempts
});

const response = await client.responses.create({
  model: "anthropic/claude-3-5-sonnet-20241022",
  input: "Hello, world!",
  agent_kwargs: {
    save_trajectory: true,
    verbosity: 20
  },
  computer_kwargs: {
    os_type: "linux",
    provider_type: "cloud"
  }
});
```

### Health Check

```typescript
const health = await client.health();
console.log(health.status); // 'healthy', 'unhealthy', 'unreachable', 'connected', 'disconnected'
```

### Cleanup

```typescript
// Clean up peer connections when done
await client.disconnect();
```

## API Reference

### AgentClient

#### Constructor

```typescript
new AgentClient(url: string, options?: AgentClientOptions)
```

- `url`: Connection URL. Supports `http://`, `https://`, or `peer://` protocols
- `options`: Optional configuration object

#### Methods

##### responses.create(request: AgentRequest): Promise<AgentResponse>

Send a request to the agent and get a response.

##### health(): Promise<{status: string}>

Check the health/connection status of the agent.

##### disconnect(): Promise<void>

Clean up resources and close connections.

### Types

#### AgentRequest

```typescript
interface AgentRequest {
  model: string;
  input: string | AgentMessage[];
  agent_kwargs?: {
    save_trajectory?: boolean;
    verbosity?: number;
    [key: string]: any;
  };
  computer_kwargs?: {
    os_type?: string;
    provider_type?: string;
    [key: string]: any;
  };
}
```

#### AgentResponse

```typescript
interface AgentResponse {
  success: boolean;
  result?: any;
  model: string;
  error?: string;
}
```

## Connection Types

### HTTP/HTTPS

Connect to a CUA agent proxy server:

```typescript
const client = new AgentClient("https://my-agent-server.com:8000");
```

### Peer-to-Peer (WebRTC)

Connect directly to another peer using WebRTC:

```typescript
const client = new AgentClient("peer://agent-proxy-peer-id");
```

The client uses PeerJS with default configuration for peer connections.

## Error Handling

```typescript
try {
  const response = await client.responses.create({
    model: "anthropic/claude-3-5-sonnet-20241022",
    input: "Hello!"
  });
  
  if (response.success) {
    console.log(response.result);
  } else {
    console.error("Agent error:", response.error);
  }
} catch (error) {
  console.error("Connection error:", error.message);
}
```

## License

MIT
