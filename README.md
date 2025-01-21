# Session Object
```ts
interface Session {
  refCode: string // Unique reference code for the session e.g. "2mfN9Xsv"
  clientOrg: string // e.g. "firstport"
  sessionType: string // "adi"
  sessionMode: string // "dev" | "stage" | "prod"
  sessionData: any // Data collected from the user (see example below)
  thread: ThreadItem[] // Thread of messages (see definition below)
  reports: any // Includes the report data emailed to property managers
  sessionMetrics: SessionMetrics // Contains metrics for the session (see definition below)
  emailSent: boolean // True once the report has been emailed
  emailThreadId?: string // Used to track email threads
}
```

---
**Example Data**
```json
// Data collected from the user
"sessionData": {
  "phone": "0786588845",
  "email": "orlando@masonsquared.com", // not required
  "lastName": "Mason",
  "firstName": "Orlando",
  "flatNumber": "21", // Not required
  "streetAddress": "427 Avonmouth Road",
  "city": "Bristol",
  "postalCode": "BS11 3QD",
  "relationshipToProperty": "tenant",
  "refCode": "2mfN9Xsv" // Only required to continue a previous session (Add updates to a previously submitted report)
},

// Draft report
"reports": {
  "adiReport": {
    "team": "green",
    "status": "draft",
    "urgency": "P2",
    "issueCategories": "leaks",
    "issueDescription": "Active leak in the kitchen ceiling, possibly from flat 2A above. The leak is continuous with clean, cold water. Currently managed with a bucket that needs emptying every 2 hours. No response from flat 2A when attempted to notify.",
  }
},

// Finished report
"reports": {
  "adiReport": {
    "team": "green",
    "status": "confirmed", // or unconfirmed if the user did not submit
    "urgency": "P2",
    "issueCategories": "leaks",
    "issueDescription": "Active leak in the kitchen ceiling, possibly from flat 2A above. The leak is continuous with clean, cold water. Currently managed with a bucket that needs emptying every 2 hours. No response from flat 2A when attempted to notify.",
    "recommendedAction": "Contact the tenant of flat 2A to investigate the leak and arrange for repairs.",
    "accessRestrictions": "None",
    "availableForContact": "yes",
    "availableForTradesperson": "yes"
  }
},

"thread": [

  {
    "formKey": "userDetails",
    "metaRole": "form",
    "submitted": false,
    "timestamp": 1736431444160
  },
  
  {
    "message": {
      "name": "Adi",
      "role": "assistant",
      "content": "Hello Orlando, I am Adi, your property assistant. How may I assist you today?"
    },
    "metaRole": "assistant",
    "timestamp": 1736431612473
  },
  
  {
    "message": {
      "name": "Orlando",
      "role": "user",
      "content": "Hi, we have a bit of a small leak in our kitchen."
    },
    "metaRole": "user",
    "timestamp": 1736431636674
  }
]
```
---
**sessionMetrics**
```ts
interface SessionMetrics {
  startTimestamp: number;
  endTimestamp: number;
  userMessageCount: number;
  assistantMessageCount: number;
  toolCallCount: number;
  totalCompletionTokens: number;
  totalPromptTokens: number;
  conversationDuration: string; // "0h 4m 27s"
  firstResponseTimeMS: number;
  averageResponseTimeMS: number;
  longestResponseTimeMS: number;
}
```
---
**ThreadItem**
```ts
type MetaRole = 'system' | 'assistant' | 'user' | 'toolCall' | 'toolOutput' | 'form' | 'report' | 'notification' | 'flagged';

interface Button {
  text: string;
  actionSequence: string;
}

interface BaseItem {
  timestamp: number;
  metaRole: MetaRole;
}

interface SystemItem extends BaseItem {
  metaRole: 'system';
  message: OpenAI.Chat.ChatCompletionSystemMessageParam;
  disabled?: boolean;
}

interface AssistantItem extends BaseItem {
  metaRole: 'assistant';
  message: OpenAI.Chat.ChatCompletionAssistantMessageParam;
  content?: string; // Displayed content (may be different from message content used in chat completion)
  disabled?: boolean;
  buttons?: Button[];
}

interface UserItem extends BaseItem {
  metaRole: 'user';
  message: OpenAI.Chat.ChatCompletionUserMessageParam;
  content?: string; // Displayed content (may be different from message content used in chat completion)
  disabled?: boolean;
}

interface ToolCallItem extends BaseItem {
  metaRole: 'toolCall';
  message: OpenAI.Chat.ChatCompletionAssistantMessageParam;
  disabled?: boolean;
}

interface ToolOutputItem extends BaseItem {
  metaRole: 'toolOutput';
  message: OpenAI.Chat.ChatCompletionToolMessageParam;
  disabled?: boolean;
}

interface FormItem extends BaseItem {
  metaRole: 'form';
  formKey: string;
  submitted: boolean;
  content?: string;
}

interface ReportItem extends BaseItem {
  metaRole: 'report';
  reportKey: string;
  submitted: boolean;
  content: string;
  buttons?: Button[];
}

interface NotificationItem extends BaseItem {
  metaRole: 'notification';
  content: string;
}

interface FlaggedItem extends BaseItem {
  metaRole: 'flagged';
  content: string;
  moderationResponse: OpenAI.Moderations.ModerationCreateResponse;
  flaggedContent: string;
}

type ThreadItem = SystemItem | AssistantItem | UserItem | ToolCallItem | ToolOutputItem | FormItem | ReportItem | NotificationItem | FlaggedItem;
```
