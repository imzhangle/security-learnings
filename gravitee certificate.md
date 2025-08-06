curl -X POST "http://localhost:8093/management/domains/{domainId}/certificates" \
  -H "Authorization: Bearer {your-access-token}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My PKCS12 Certificate",
    "type": "pkcs12",
    "configuration": {
      "content": "'$(base64 -w 0 ./my-cert.p12)'",
      "password": "yourP12Password",
      "alias": "my-alias"
    }
  }'
