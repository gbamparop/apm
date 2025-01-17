The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to
be interpreted as described in
[RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

## Data sanitization

### `sanitize_field_names` configuration

Sometimes it is necessary to sanitize, i.e., remove,
sensitive data sent to Elastic APM.

This config accepts a list of wildcard patterns of field names which control
how an agent will sanitize data.

|                |   |
|----------------|---|
| Type           | `List<`[`WildcardMatcher`](../../tests/agents/json-specs/wildcard_matcher_tests.json)`>` |
| Default        | `password, passwd, pwd, secret, *key, *token*, *session*, *credit*, *card*, *auth*, set-cookie, *principal*` |
| Dynamic        | `true` |
| Central config | `true` |

#### Configuration

Agents MUST provide a minimum default configuration of

    [ 'password', 'passwd', 'pwd', 'secret', '*key', '*token*', '*session*',
      '*credit*','*card*', '*auth*', 'set-cookie', '*principal*' ]

for the `sanitize_field_names` configuration value.  Agent's MAY include the
following extra fields in their default configuration to avoid breaking changes

    ['pw','pass','connect.sid']

## Sanitizing Values

If a payload field's name (a header key, a form key) matches a configured
wildcard, that field's _value_ MUST be redacted and the key itself
MUST still be reported in the agent payload. Agents MAY choose the string
they use to replace the value so long as it's consistent and does not reveal
the value it has replaced. The replacement string SHOULD be `[REDACTED]`.

Fields that MUST be sanitized are:
- HTTP Request and Response headers (except [HTTP/2 pseudo-headers](https://datatracker.ietf.org/doc/html/rfc7540#section-8.1.2.3) which SHOULD NOT be redacted), and
- form fields in an `application/x-www-form-urlencoded` request body.

The query string and other captured request bodies (such as `application/json`)
SHOULD NOT be sanitized.

Agents SHOULD NOT sanitize fields based on the _value_ of a particular field.
