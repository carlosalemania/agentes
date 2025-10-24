# Serverless Architect - Examples

---

## Example: Complete Serverless API

```bash
# Deploy with Serverless Framework
serverless deploy --stage prod

# Invoke function locally
serverless invoke local --function getItem --path test-event.json

# View logs
serverless logs --function getItem --tail
```

```javascript
// Lambda handler with middleware pattern
const middy = require('@middy/core');
const httpErrorHandler = require('@middy/http-error-handler');
const validator = require('@middy/validator');
const cors = require('@middy/http-cors');

const baseHandler = async (event) => {
  const { id } = event.pathParameters;

  const item = await dynamodb.get({
    TableName: process.env.TABLE_NAME,
    Key: { id }
  }).promise();

  return {
    statusCode: 200,
    body: JSON.stringify(item.Item)
  };
};

const handler = middy(baseHandler)
  .use(validator({
    inputSchema: {
      type: 'object',
      properties: {
        pathParameters: {
          type: 'object',
          properties: {
            id: { type: 'string', minLength: 1 }
          },
          required: ['id']
        }
      }
    }
  }))
  .use(cors())
  .use(httpErrorHandler());

module.exports = { handler };
```

---

## Example: EventBridge Pattern

```yaml
# EventBridge rule
OrderCreatedRule:
  Type: AWS::Events::Rule
  Properties:
    EventBusName: default
    EventPattern:
      source:
        - com.myapp.orders
      detail-type:
        - OrderCreated
    Targets:
      - Arn: !GetAtt ProcessOrderFunction.Arn
        Id: ProcessOrderTarget
      - Arn: !GetAtt SendEmailFunction.Arn
        Id: SendEmailTarget
```

---

**Versión:** 1.0.0
