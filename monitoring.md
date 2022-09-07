one way of node monitoring is to regularly request JSON RPC endpoint and send results back to one of contact points, for example TelegramBot.
again we would need a script that would execute everything and cron job to run the script on some schedule

script below will provide info about
- stake
- shards
- block produced/expected
- chunks produced/expected
- node state
```
#!/bin/bash
CHANNEL_ID=""
BOT_TOKEN=""
URL="https://api.telegram.org/bot${BOT_TOKEN}/sendMessage"
ACCOUNT_ID=

validator_info=$(curl -s -d '{"jsonrpc": "2.0", "method": "validators", "id": "dontcare", "params": [null]}' -H 'Content-Type: application/json' 127.0.0.1:3030 | jq -c '.result.current_validators[] | select(.account_id=="[ACCOUNT_ID]")')

is_slashed=$(echo $validator_info | jq .is_slashed)
validator=$(echo $validator_info | jq .account_id)
stake=$(echo $validator_info | jq .stake)
shards=$(echo $validator_info | jq .shards[])

num_expected_blocks=$(echo $validator_info | jq .num_expected_blocks)
num_expected_chunks=$(echo $validator_info | jq .num_expected_chunks)
num_produced_blocks=$(echo $validator_info | jq .num_produced_blocks)
num_produced_chunks=$(echo $validator_info | jq .num_produced_chunks)

is_validator="false"

if [[ $validator ]]
    then
    is_validator="true"
fi

curl -s -X POST $URL \
-d chat_id=${CHANNEL_ID} \
-d parse_mode="Markdown" \
-d text="*${ACCOUNT_ID}* %0Ais slashed: ${is_slashed}%0Ais validating: ${is_validator}%0Astake: ${stake}%0Ashards: ${shards}%0Aexpected blocks: ${num_expected_blocks}%0Aproduced blocks: ${num_produced_blocks}%0Aexpected chunks: ${num_expected_chunks}%0Aproduced chunks: ${num_produced_chunks}"
```

and cron job might look like `0 */4 * * * /home/near-monitoring/monitor.sh` assuming `monitor.sh` directory
