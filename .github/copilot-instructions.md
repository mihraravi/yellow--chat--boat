# Watson Banking Chatbot - Copilot Instructions

## Architecture Overview

This is an **IBM Watson-based conversational banking chatbot** connecting three Watson AI services through a Node.js/Express backend.

### Service Architecture

The chatbot pipeline has three sequential stages:

1. **Watson Assistant** (`app.js` /api/message) - Receives user input and manages dialog flow
2. **Watson Natural Language Understanding (NLU)** - Enriches context with entity detection (Location, sentiment, keywords) before passing to Assistant
3. **Watson Discovery** - Provides FAQ answers via passage retrieval when Assistant requests action `disco`

### Context Flow Pattern

- **User input** → NLU analysis (extracts location entities) → **Assistant receives enriched context** → Assistant responds/requests action
- If Assistant output contains `generic[].response_type == "user_defined"` with `user_defined.action`, the app triggers banking service lookups or Discovery queries
- **Locale support**: Application switches between `EN_IN` (India) and `EN_US` (USA) via `LOCALE` env var, which controls skill file, Discovery docs, and banking service data

### Banking Services Layer

Two parallel service implementations:
- `banking_services.js` - Indian account data (krishna@example.com, Mumbai address)
- `banking_services_us.js` - US account data (same interface, different values)

Both export: `getPerson()`, `getAccountInfo()`, `getBeneficiaryInfo()`, `getTransactions()`, `update5Transactions()`

## Key Integration Patterns

### 1. Message Endpoint (`POST /api/message`)

```javascript
// Request body structure
{ 
  input: { text: "user query" },
  context: { /* Watson context object */ }
}

// Response is raw Watson Assistant output with:
// - output.text (assistant reply)
// - output.generic[].response_type (determines action handling)
// - context (persisted by client, returned in next request)
```

**Important**: Context is **client-managed** - the UI maintains state between turns.

### 2. Action Handling in Assistant Output

Assistant can request actions via `user_defined` response:
- `action: "balance"` → `LOOKUP_BALANCE` → calls `bankingServices.getAccountInfo()`
- `action: "transactions"` → `LOOKUP_TRANSACTIONS` → calls `bankingServices.getTransactions()`
- `action: "5transactions"` → `LOOKUP_5TRANSACTIONS` → calls `bankingServices.update5Transactions()`
- `action: "disco"` → `DISCOVERY_ACTION` → queries Discovery with passage retrieval for FAQs

See `checkForLookupRequests()` in app.js for exact routing.

### 3. PAN Masking Pattern

Input text is scanned for PAN numbers (regex: `/^([a-zA-Z]){5}([0-9]){4}([a-zA-Z]){1}?$/`) and replaced with `1111111111` before sending to Watson (privacy compliance).

## Development Workflows

### Local Setup
```bash
npm install                    # Install dependencies
cp env.sample .env            # Configure Watson credentials
npm start                      # Starts Express on port 3000 (or $PORT)
```

### Testing
```bash
npm test                       # Runs all tests (unit + eslint + jshint)
npm run unit                   # Unit tests only (mocha with Istanbul coverage)
npm run eslint                 # ESLint on *.js, lib/**, test/**
npm run eslint-fix             # Auto-fix linting issues
npm run jshint                 # JSHint on public/**
```

### Credential Management

Three deployment models supported:

1. **IBM Cloud (default)**: Use `ASSISTANT_APIKEY`, `DISCOVERY_APIKEY`, `NATURAL_LANGUAGE_UNDERSTANDING_APIKEY`
2. **Cloud Pak for Data (CP4D)**: Use `ASSISTANT_AUTH_TYPE=cp4d` + username/password (see env.sample)
3. **Legacy CONVERSATION_ prefix**: Code handles backward compatibility (`app.js` lines 40-54)

Environment variables are loaded via `dotenv` in app.js.

## File Organization

- `app.js` - Main Express app, message routing, Watson service orchestration
- `server.js` - Port binding entrypoint (delegates to app.js)
- `lib/watson-assistant-setup.js` - Creates/validates skill workspace from `data/conversation/workspaces/banking_*.json`
- `lib/watson-discovery-setup.js` - Initializes Discovery collection with FAQ documents from `data/discovery/docs/`
- `public/js/api.js` - XMLHttpRequest wrapper for `/api/message` endpoint
- `public/js/conversation.js` - UI message display and input handling
- `test/unit/test.banking_services.js` - Unit tests for account/person/beneficiary lookups

## Important Constraints & Conventions

1. **Watson SDK Version**: `ibm-watson@5.2.1` - Check API compatibility before upgrading
2. **Hardcoded Customer ID**: `7829706` used in `/api/message` - banking service data is mocked by ID, not retrieved
3. **Context Keys**: Assistant puts enriched data in context: `nlu_output`, `Location`, `person` object
4. **Skill JSON Format**: Dialog nodes exported as `dialog_nodes` are auto-converted to `dialogNodes` format (backward compatibility in app.js:108-110)
5. **No Database**: All account/transaction data is in-memory from `banking_services.js` objects
6. **Single User Experience**: No multi-user session management; all requests use same customer ID

## Common Debugging Points

- **Assistant not responding**: Check `skillID` initialization (assistant-setup.js may still be running)
- **Discovery not working**: Verify documents are in `data/discovery/docs/{locale}/` directory
- **Location detection failing**: NLU analysis log shows entities - check `parameters.features.entities` matches Watson NLU v2019-07-12 schema
- **Action lookups returning 500**: `checkForLookupRequests()` error handling will log the specific banking service error
