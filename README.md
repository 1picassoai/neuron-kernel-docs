# Mirror-Code: High-Fidelity IaC Simulation Engine

The Mirror-Code system allows developers to run "Virtual-Shadow" simulations against synthetic workloads. This image contains the core Neuron Kernel used for high-throughput state management and persistence.
The Neuron Kernel acts like a persistent "external brain" for AI agents, providing a stable place to store and recall their operational progress so they never lose their place, even if the agent's process crashes or restarts.

## 🚀 Quickstart: Running the Local Node
To try out the engine on your local machine (Mac/Linux/Windows), follow these steps.

### Minimum Requirements
* **RAM**: 8GB (Minimum) / 16GB (Recommended)
* **Runtime**: Docker / Containerd

### Execution
1. Create a directory for persistent data:
   `mkdir -p ~/mirror-data && chmod 777 ~/mirror-data`
2. docker pull picassoai/neuron-kernel:v1.2
3. Launch the kernel:
   ```bash
  docker run -d \
  --name mirror-node \
  --restart unless-stopped \
  -p 8080:8080 \
  -e OPTS="-Xmx4g -Xms4g" \
  -v ~/mirror-data:/app/data \
  --health-cmd="curl -f http://localhost:8080/state/health || exit 1" \
  --health-interval=10s \
  --health-retries=3 \
  picassoai/neuron-kernel:v1.2
4. Finally run - `docker inspect --format='{{json .State.Health.Status}}' mirror-node`

###🛠 API Interface
Once the node is running, you can interact with the engine via REST.

POST /state (Sync Data)
Injects simulation data into the engine. All 4 fields are required:

id: Session ID

status: Operational mode

data: Metric value

step: Sequence index

Example:
curl -X POST http://localhost:8080/state -H "Content-Type: application/json" -d '{"id":"test-01","status":"active","data":"1.0","step":"1"}'

GET /state (Retrieve State)
Retrieves the current committed state from the engine.
curl http://localhost:8080/state

# Example: Agentic SDK Integration for Mirror-Code
from agent_sdk import Agent, Task, Tool
import requests

# 1. Define the Mirror-Code Tool
class MirrorKernelTool(Tool):
    name = "kernel_state_sync"
    description = "Synchronizes agent logic with the high-fidelity state engine."

    def _run(self, payload: dict):
        """
        Payload must contain: id, status, data, step
        """
        url = "http://localhost:8080/state"
        response = requests.post(url, json=payload)
        return response.status_code

# 2. Initialize the Infrastructure Agent
simulation_agent = Agent(
    role="Infrastructure Shadow Controller",
    goal="Maintain 100% state parity between local and cloud nodes.",
    backstory="You are an autonomous auditor for Synthetic 2026 Workloads.",
    tools=[MirrorKernelTool()],
    verbose=True
)

# 3. Define the Agentic Task
sync_task = Task(
    description="Analyze the current simulation step and sync with the kernel.",
    agent=simulation_agent,
    expected_output="A '200 OK' synchronization confirmation."
)

# 4. Execute (Agent handles the JSON schema automatically)
simulation_agent.execute(sync_task)

# How the Agent Interacts with your Engine
Observation: The agent queries the GET /state endpoint to see where the last "Step" ended.

Reasoning: The agent determines if the next "Synthetic Workload" is ready to be deployed.

Action: The agent calls the POST /state endpoint with the mandatory 4-field JSON body.

Verification: The agent confirms the "Mirror" is persistent before proceeding to the next task.
