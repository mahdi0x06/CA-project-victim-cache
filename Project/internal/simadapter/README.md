# Simulation adapter

`simadapter` is the Akita runtime boundary used by every command.

It creates:

- an Akita serial engine;
- a request-driver component and port;
- a hierarchy-executor component and port;
- typed Akita request and response messages;
- an Akita direct connection; and
- scheduled access-completion events.

The hierarchy executor calls the unchanged functional `System.Access` exactly
once per request. The driver waits for that response before sending the next
request, preserving all pre-integration results while making Akita responsible
for request transport and event scheduling.

See the repository root file `AKITA_INTEGRATION.md` for the complete design.
