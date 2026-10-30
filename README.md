# strands-aerospike

Aerospike-backed persistence for the [Strands Agents](https://github.com/strands-agents/harness-sdk) Python SDK.

The package provides three Strands persistence backends:

- **`AerospikeStorage`** implements `strands.storage.Storage` for byte values, with prefix listing and large-value chunking.
- **`AerospikeSessionManager`** implements session persistence for sessions, agents, messages, and multi-agent state.
- **`AerospikeMemoryStore`** implements `strands.memory.MemoryStore` for named memory stores, with lexical search.

## Install

```bash
pip install strands-aerospike
```

Requires Python 3.10 or later and an Aerospike namespace. Constructors use
`127.0.0.1:3000` by default; pass `hosts=[("host", 3000)]` to connect to another
cluster, or pass an existing Aerospike client with `client=`.

## Use `AerospikeStorage`

```python
import asyncio

from strands_aerospike import AerospikeStorage


async def main() -> None:
    storage = AerospikeStorage(namespace="test")
    await storage.write("agent/state", b"ready")
    state = await storage.read("agent/state")
    print(state)


asyncio.run(main())
```

## Use with a Strands agent

```python
import asyncio

from strands import Agent
from strands.memory import MemoryManager
from strands_aerospike import AerospikeMemoryStore, AerospikeSessionManager


async def main() -> None:
    session_manager = AerospikeSessionManager(session_id="my-session", namespace="test")
    memory_store = AerospikeMemoryStore(name="agent-memory", namespace="test")
    await memory_store.initialize()

    agent = Agent(
        session_manager=session_manager,
        memory_manager=MemoryManager(stores=[memory_store]),
    )
    await agent.invoke_async("Remember that I prefer concise answers.")


asyncio.run(main())
```
