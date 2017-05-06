---
post_title: Logging API Examples
feature_maturity: preview
menu_order: 4
---

The following examples exercise the Logging API using [Bash](https://www.gnu.org/software/bash/), [Curl](https://curl.haxx.se/), and [jq](https://stedolan.github.io/jq/).

The [DC/OS CLI](/docs/1.9/cli/) must be installed, configured, and logged in. Then, extract `DCOS_URL` and `DCOS_AUTH_TOKEN` from the DC/OS CLI:

```
DCOS_URL="$(dcos config show core.dcos_url)"
DCOS_URL="${DCOS_URL%/}" # strip trailing slash, if present
DCOS_AUTH_TOKEN="$(dcos config show core.dcos_acs_token)"
```

# Tail

Get the last 10 entries from the journal and follow new events:

```
curl -k -H "Authorization: token=${DCOS_AUTH_TOKEN}" "${DCOS_URL}/system/v1/logs/v1/stream/?skip_prev=10"
```

# Range

Skip 100 entries from the beginning of the journal and return the next 10 entries:

```
curl -k -H "Authorization: token=${DCOS_AUTH_TOKEN}" "${DCOS_URL}/system/v1/logs/v1/range/?skip_next=100&limit=10"
```

# Plain Text

Skip 200 entries from the beginning of the journal and return the next entry in plain text:

```
curl -k -H "Accept: text/plain" -H "Authorization: token=${DCOS_AUTH_TOKEN}" "${DCOS_URL}/system/v1/logs/v1/range/?skip_next=200&limit=1"
```

# JSON

Skip 200 entries from the beginning of the journal and return the next entry in JSON:

```
curl -k -H "Accept: application/json" -H "Authorization: token=${DCOS_AUTH_TOKEN}" "${DCOS_URL}/system/v1/logs/v1/range/?skip_next=200&limit=1"
```

# Event Stream

Skip 200 entries from the beginning of the journal and return the next entry as an event stream:

```
curl -k -H "Accept: text/event-stream" -H "Authorization: token=${DCOS_AUTH_TOKEN}" "${DCOS_URL}/system/v1/logs/v1/range/?skip_next=200&limit=1"
```

# Event Stream Cursor

Get all logs after a specific cursor and follow new events:

```
# Get the 10th line in json and parse its cursor
CURSOR="$(curl -k -H "Accept: application/json" -H "Authorization: token=${DCOS_AUTH_TOKEN}" "${DCOS_URL}/system/v1/logs/v1/range/?skip_next=9&limit=1" | jq -r ".cursor")"

# Follow the stream in plain text starting at the 11th line using the cursor
curl -k -H "Authorization: token=${DCOS_AUTH_TOKEN}" "${DCOS_URL}/system/v1/logs/v1/stream/" --get --data-urlencode "cursor=${CURSOR}"
```

The cursor must be URL encoded.

# Range Cursor

Get 1,000 log lines, 100 at a time:

```
TIMES=10 # number of times to call the endpoint
LINES=100 # number of log lines to retrieve per call
CURSOR="" # first call uses an empty cursor to start from the beginning
for i in $(seq 1 ${TIMES}); do
  BUFFER="$(curl -k -H "Accept: application/json" -H "Authorization: token=${DCOS_AUTH_TOKEN}" "${DCOS_URL}/system/v1/logs/v1/range/?limit=${LINES}" --get --data-urlencode "cursor=${CURSOR}")"
  echo "${BUFFER}"
  CURSOR="$(echo "${BUFFER}" | jq -r ".cursor" | tail -1)"
done
```

Each call uses the cursor of the last line from the previous call. The cursor must be URL encoded.
