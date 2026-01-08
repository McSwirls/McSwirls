# StreamElements "No" Command

A fun Twitch chat command that responds with random creative reasons for saying "no" to anything.

## How It Works

When viewers type `!no` in chat, the bot responds with humorous rejection messages like:
- "Life is short, and my nap list is long."
- "My procrastination coach said I'm not ready for actual tasks yet."
- "I'd rather search for the purpose of life than attend."

## Technical Setup

### Components

1. **API Source**: [No-as-a-Service](https://github.com/islamlists/No-as-a-service) by islamlists
   - Endpoint: `https://naas.isalman.dev/no`
   - Returns JSON with a random "no" reason

2. **Cloudflare Worker**: Acts as a proxy/parser
   - Fetches the JSON from the API
   - Extracts just the `reason` field
   - Returns plain text (no JSON formatting)

3. **StreamElements Bot**: Delivers the response to Twitch chat
   - Command: `!no`
   - Fetches from the Cloudflare Worker
   - Displays the plain text response

### Architecture Flow

```
Viewer types !no
    ↓
StreamElements Bot
    ↓
Cloudflare Worker (your-worker.workers.dev)
    ↓
No-as-a-Service API (naas.isalman.dev)
    ↓
Returns JSON: {"reason": "..."}
    ↓
Worker extracts "reason"
    ↓
Returns plain text to StreamElements
    ↓
Bot posts message in Twitch chat
```

## Cloudflare Worker Code

```javascript
export default {
  async fetch(request, env, ctx) {
    try {
      const response = await fetch('https://naas.isalman.dev/no');
      const data = await response.json();
      
      return new Response(data.reason || 'No reason provided', {
        headers: {
          'Content-Type': 'text/plain',
          'Access-Control-Allow-Origin': '*'
        }
      });
    } catch (error) {
      return new Response('Error fetching response', {
        status: 500,
        headers: { 'Content-Type': 'text/plain' }
      });
    }
  }
};
```

## StreamElements Command Setup

**Command Name:** `!no`

**Response:**
```
${urlfetch https://your-worker.your-subdomain.workers.dev}
```

**Settings:**
- Cooldown: 10 seconds (recommended to respect API rate limits)
- User Level: Everyone
- Cost: 0 points

## Rate Limits

- No-as-a-Service API: 10 requests per minute per IP
- Cloudflare Workers Free Tier: 100,000 requests per day
- StreamElements command cooldown handles rate limiting

## Credits

- [No-as-a-Service API](https://github.com/islamlists/No-as-a-service) by islamlists
- Cloudflare Workers for serverless hosting
- StreamElements for bot functionality

## License

This setup is provided as-is for personal use. The No-as-a-Service API is subject to its own terms of use.
