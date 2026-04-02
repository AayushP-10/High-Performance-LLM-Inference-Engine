LLM INference Engine
Efficient handling of machine learning inference requests often relies on batching multiple inputs together. Instead of processing each request individually, combining them into a single batch can significantly improve performance, particularly on GPUs.
This project provides a simple way to implement dynamic batching in Python-based web frameworks such as FastAPI. Incoming requests are grouped together and processed as a batch, enabling better utilization of compute resources and higher throughput. In testing, this approach achieved up to 2.5× higher throughput compared to non-batched execution.
---
Overview
The dynamic batching system collects incoming requests and processes them based on two conditions:
The batch reaches a predefined maximum size
A specified delay threshold is exceeded
This allows the system to balance latency and throughput, ensuring efficient execution without introducing excessive delays.
---
Example Usage
To enable dynamic batching, define a custom inference model by extending the `InferenceModel` base class. Initialize your ML model in the constructor and implement the batch processing logic inside the `infer` method.
```python
from typing import Any
from fast_dynamic_batcher.inference_template import InferenceModel

class DemoModel(InferenceModel):
    def __init__(self):
        super().__init__()
        # Initialize your ML model here

    def infer(self, inputs: list[Any]) -> list[Any]:
        # Run inference on the batch
        outputs = ...  # Replace with actual model outputs
        return outputs
```
Next, initialize the dynamic batcher within the FastAPI application lifecycle:
```python
from contextlib import asynccontextmanager
from anyio import CapacityLimiter
from anyio.lowlevel import RunVar
from fast_dynamic_batcher.dyn_batcher import DynBatcher
from fastapi import FastAPI

@asynccontextmanager
async def lifespan(app: FastAPI):
    RunVar("_default_thread_limiter").set(CapacityLimiter(16))
    global dyn_batcher
    dyn_batcher = DynBatcher(
        DemoModel,
        max_batch_size=8,
        max_delay=0.1
    )
    yield
    dyn_batcher.stop()

app = FastAPI(lifespan=lifespan)

@app.post("/predict/")
async def predict(input_model: YourInputPydanticModel):
    return await dyn_batcher.process_batched(input_model)
```
---
How It Works
Incoming API requests are placed into a temporary queue
The system waits until either:
the number of requests reaches `max_batch_size`, or
`max_delay` seconds have passed
Once one of these conditions is satisfied, the batch is processed
The batch is passed to the model’s `infer` method
Outputs are returned individually to each request
This mechanism allows batch sizes to adapt dynamically based on traffic, improving efficiency without manual tuning.
---
Configuration
The batching behavior can be customized using:
max_batch_size: Maximum number of requests in a batch
max_delay: Maximum waiting time before processing
These parameters allow tuning for different system goals:
Lower delay improves latency
Larger batch sizes improve throughput
---
Installation
```bash
pip install fast_dynamic_batcher
```
---
Performance Evaluation
The system was tested against a baseline without batching on both GPU and CPU environments.
Hardware	No Batching	Dynamic Batch (size = 16)
Colab T4 GPU	7.65s	3.07s
Intel i7-1250U CPU	117.10s	88.47s
Observations
GPUs benefit significantly from batching due to parallel execution
Achieved nearly 2.5× speedup on GPU workloads
CPU performance improved by approximately 1.3×
---
Key Benefits
Increased inference throughput
Improved hardware utilization
Reduced overhead per request
Easy integration with FastAPI
Flexible control over latency and efficiency tradeoffs
---
Summary
This project demonstrates how dynamic batching can be integrated into modern inference pipelines with minimal complexity. By intelligently grouping requests and processing them together, it enables scalable and efficient deployment of machine learning models in real-world applications.
