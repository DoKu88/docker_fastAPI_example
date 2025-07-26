# Network Experiment

A simple microservices experiment demonstrating inter-container communication using FastAPI and Python. The project consists of two services:

- **main**: A Python service that generates large numpy arrays and sends them for processing
- **manip_seg**: A FastAPI service that receives arrays, processes them, and returns the results

## Architecture

```
┌─────────────┐    HTTP POST    ┌─────────────┐
│    main     │ ──────────────► │  manip_seg  │
│             │                 │             │
│ - Generates │                 │ - Receives  │
│   arrays    │                 │   arrays    │
│ - Sends to  │                 │ - Processes │
│   manip_seg │                 │ - Returns   │
└─────────────┘                 │   results   │
                                └─────────────┘
```

## Quick Start with Docker Compose

1. **Clone and navigate to the project:**
   ```bash
   cd network_experiment
   ```

2. **Run with Docker Compose:**
   ```bash
   docker-compose up --build
   ```

3. **View the results:**
   The main service will generate a 1000x1000 array, send it to manip_seg for processing (multiplied by 2), and display the results.

## Running Without Docker Compose

### Option 1: Local Python (No Docker)

#### Prerequisites
- Python 3.8+
- pip

#### Step 1: Install Dependencies

**For manip_seg service:**
```bash
cd manip_seg
pip install fastapi uvicorn numpy
```

**For main service:**
```bash
cd main_container
pip install requests numpy
```

#### Step 2: Start the manip_seg Service

In one terminal:
```bash
cd manip_seg
python server.py
```

The service will start on `http://localhost:8000`

#### Step 3: Run the Main Service

In another terminal:
```bash
cd main_container
python main.py
```

**Note:** You'll need to modify the URL in `main.py` from `http://manip_seg:8000` to `http://localhost:8000` when running locally.

#### Step 4: Verify It's Working

- The manip_seg service should show: `INFO: Uvicorn running on http://0.0.0.0:8000`
- The main service should show the array processing results

### Option 2: Manual Docker (No Docker Compose)

#### Prerequisites
- Docker installed

#### Step 1: Build the Images

```bash
# Build manip_seg image
docker build -t network_experiment-manip_seg ./manip_seg

# Build main image
docker build -t network_experiment-main ./main_container
```

#### Step 2: Create Docker Network

```bash
docker network create dockerNetwork1
```

#### Step 3: Run manip_seg Container

```bash
docker run --rm --name manip_seg --network dockerNetwork1 -p 8000:8000 network_experiment-manip_seg
```

#### Step 4: Run Main Container

In another terminal:
```bash
docker run --rm --name main --network dockerNetwork1 network_experiment-main
```

**Note:** The containers can communicate using service names (e.g., `http://manip_seg:8000`) because they're on the same network.

## API Endpoints

### manip_seg Service

- `GET /` - Health check endpoint (returns `{"status": "ok"}`)
- `POST /process` - Process numpy arrays
  - **Request:** `{"data": "base64_encoded_array"}`
  - **Response:** `{"result": "base64_encoded_processed_array"}`

## Data Flow

1. **main** generates a 1000x1000 numpy array with random values
2. **main** serializes the array to base64 and sends it to **manip_seg**
3. **manip_seg** deserializes the array, multiplies it by 2, and serializes the result
4. **manip_seg** returns the processed array to **main**
5. **main** deserializes and displays the results

## Troubleshooting

### Connection Issues
- **Docker Compose:** Ensure both containers are running with `docker ps`
- **Local:** Make sure manip_seg is started before main

### Port Conflicts
- **Docker Compose:** Port 8000 is exposed to host
- **Local:** Ensure port 8000 is available on your machine

### Service Discovery
- **Docker Compose:** Services can reach each other by service name (`manip_seg`)
- **Local:** Use `localhost` or `127.0.0.1`

## Project Structure

```
network_experiment/
├── docker-compose.yml          # Docker Compose configuration
├── main_container/
│   ├── Dockerfile             # Main service container
│   └── main.py               # Main service logic
├── manip_seg/
│   ├── Dockerfile            # manip_seg service container
│   └── server.py            # FastAPI server
└── README.md                 # This file
```

## Development

### Adding New Processing Logic
Modify the processing logic in `manip_seg/server.py`:

```python
# Example: Add more complex processing
processed_array = array * 2 + 1  # Multiply by 2 and add 1
```

### Adding New Endpoints
Add new endpoints to `manip_seg/server.py`:

```python
@app.post("/custom_process")
async def custom_process(request: Request):
    # Your custom logic here
    pass
```

### Scaling
To run multiple instances of manip_seg:

```bash
# Docker Compose
docker-compose up --scale manip_seg=3

# Local (different ports)
python server.py --port 8001  # Terminal 1
python server.py --port 8002  # Terminal 2
python server.py --port 8003  # Terminal 3
```

## License

This project is open source and available under the MIT License. 