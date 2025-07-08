```shell
#!/bin/sh

# Set your variables here
BASE_URL=""
USERNAME=""
PASSWORD=""

auth_response=$(curl -s -X POST "$BASE_URL/api/collections/users/auth-with-password" \
  -H "Content-Type: application/json" \
  -d "{\"identity\":\"$USERNAME\", \"password\":\"$PASSWORD\"}")

token=$(echo "$auth_response" | grep -o '"token"[[:space:]]*:[[:space:]]*"[^"]*"' | sed -E 's/.*"token"[[:space:]]*:[[:space:]]*"([^"]*)".*/\1/')

if [ -z "$token" ]; then
  echo "Failed to get token from auth response:"
  echo "$auth_response"
  exit 1
fi

echo "Auth token: $token"
echo
```

```shell
curl -X POST "BASE_URL/api/collections/users/auth-with-password" \
  -H "Content-Type: application/json" \
  -d '{"identity":"USERNAME","password":"PASSWORD"}'
```
