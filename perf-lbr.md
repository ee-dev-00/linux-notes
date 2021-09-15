Using perf with last branch record

### Example

    perf record --call-graph lbr -F 10000 -p 10072 -- sleep 3
