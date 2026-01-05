# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Amazon Connect Custom CCP - A custom Contact Control Panel dashboard for Amazon Connect contact centers. Two-part architecture: AWS CDK infrastructure (Node.js) and React frontend.

## Common Commands

### CDK Infrastructure (ccp-cdk-infra/)
```bash
cd ccp-cdk-infra
npm install
npm run bootstrap   # First-time CDK setup
npm run deploy      # Deploy stack to AWS
npm run diff        # Compare deployed vs current
npm run synth       # Generate CloudFormation template
```

### React UI (ccp-ui/)
```bash
cd ccp-ui
npm install
npm start           # Development server
npm run build       # Production build
```

### Environment Setup (This Project)
```bash
export AWS_REGION=ap-northeast-1
export AWS_PROFILE=main
```

## Architecture

```
ccp-cdk-infra/          # AWS CDK infrastructure
├── lib/
│   ├── ccp-stack.js    # Main CDK stack (all AWS resources defined here)
│   └── lambdaCode/     # Lambda functions (13 total)
├── config/
│   └── project-config.json  # Naming conventions
└── package.json

ccp-ui/                 # React frontend
├── src/
│   ├── App.jsx         # Router + NorthStar theme setup
│   ├── config.js       # API endpoints (CONNECT_NAME, API_GATEWAY_ID, CF_DISTRIBUTION_URL)
│   ├── hooks.js        # Amazon Connect subscription hooks
│   ├── lib.js          # Utility functions
│   └── components/
│       ├── Dashboard.jsx      # Main 2-column layout
│       ├── ConnectCCP.jsx     # Amazon Connect Streams integration
│       ├── CustomerInfo.jsx   # Customer details + call controls
│       ├── ActionsSection.jsx # Start/Stop/Pause/Resume/Transfer
│       └── StatisticsModal.jsx # Daily metrics display
└── package.json
```

## Key Integration Points

**Amazon Connect Streams SDK** (`amazon-connect-streams` 2.3.1): Core SDK for CCP functionality. Custom hooks in `hooks.js` subscribe to contact state changes:
- `useContact` - Contact subscription
- `useConnected` - Call connect event
- `useDestroy` - Call end event
- `useCallCompleted` - Call completion/ACW event

**DynamoDB Tables**:
- `AgentInfo` - Daily agent activity, intent selections (PK: agentName, SK: date)
- `CallIntents` - Available intents per queue (PK: queueName)

**API Gateway**: 13 REST endpoints connecting to Lambda functions for metrics and call management.

## Adding New Lambda Functions

1. Create directory in `ccp-cdk-infra/lib/lambdaCode/<function-name>/index.js`
2. Add to `dependsOnConnectPolicy` array in `ccp-stack.js` if it needs Connect permissions
3. Add to `allresources` array: `{ Type: '<METHOD>', Name: '<resource>', Lambda: '<function-name>' }`

## Configuration

**Before deployment**:
- Set Amazon Connect instance ID as SSM parameter with key `connect-id`
- Update queue names/intents in `ccp-cdk-infra/lib/lambdaCode/populateDB/`
- Update company name in `project-config.json`

**For local development**:
- Update `CONNECT_NAME`, `API_GATEWAY_ID`, `CF_DISTRIBUTION_URL` in `ccp-ui/src/config.js`

## Important Notes

- This project uses `ap-northeast-1` region with AWS profile `main`
- `config.js` API_HOST needs to be updated to `ap-northeast-1` (currently hardcoded to `us-west-2`)
- Lambda functions must return `Access-Control-Allow-Origin` and `Access-Control-Allow-Credentials` headers
- Uses AWS CDK v1 (not v2)
- React frontend uses AWS Northstar design system and @awsui/components-react
