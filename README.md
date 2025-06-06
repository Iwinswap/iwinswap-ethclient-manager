# Iwinswap ETH Client Manager

A robust, production-ready Go package for managing connections to multiple Ethereum nodes. It provides health monitoring, latency tracking, and intelligent client selection to ensure high availability and performance for decentralized applications.

---

## Overview

Interacting with Ethereum nodes can be unreliable—nodes may fall out of sync, become unresponsive, or introduce latency. `iwinswap-ethclient-manager` abstracts away this complexity by wrapping one or more `go-ethereum` clients with a management layer that continuously monitors health and performance.

The package provides:

* `Client`: A wrapper around a single `ethclient.Client` that adds health checks, latency tracking, concurrency limits, and graceful lifecycle management.
* `ClientManager`: A high-level manager that maintains a pool of `Client` instances and selects the best one for your RPC calls based on health and latency.

---

## Features

* **Automatic Health Monitoring**
  Periodically checks the status of each node by fetching the latest block number.

* **Latency Tracking**
  Records response time for each health check.

* **Intelligent Client Selection**

  * `GetClient()`: Returns a healthy, synced client using round-robin rotation.
  * `GetPreferredClient()`: Selects a low-latency, healthy client from a preferred pool.

* **Concurrency Limiting**
  Each client limits concurrent RPC calls for stability under load.

* **Graceful Shutdown**
  Uses `context.Context` for safe termination of goroutines and connections.

* **Customizable Configuration**
  Fine-tune polling intervals, timeouts, and concurrency per client.

---

## Installation

```bash
go get github.com/Iwinswap/iwinswap-ethclient-manager
```

---

## Usage

### Basic Example

```go
package main

import (
	"context"
	"fmt"
	"log"
	"time"

	"github.com/Iwinswap/iwinswap-ethclient-manager/clientmanager"
)

func main() {
	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	endpoints := []string{
		"https://eth.public-node.com",
		"https://ethereum.publicnode.com",
		"https://rpc.ankr.com/eth",
	}

	managerConfig := &clientmanager.ClientManagerConfig{
		ClientConfig: &clientmanager.ClientConfig{},
	}

	manager, err := clientmanager.NewClientManager(ctx, endpoints, managerConfig)
	if err != nil {
		log.Fatalf("Failed to create client manager: %v", err)
	}
	defer manager.Close()

	log.Println("Waiting for clients to become healthy...")
	time.Sleep(6 * time.Second)

	client, err := manager.GetClient()
	if err != nil {
		log.Fatalf("Failed to get a healthy client: %v", err)
	}

	blockNumber, err := client.BlockNumber(context.Background())
	if err != nil {
		log.Fatalf("Failed to get latest block number: %v", err)
	}
	fmt.Printf("Latest block number: %d\n", blockNumber)

	preferredClient, err := manager.GetPreferredClient()
	if err != nil {
		log.Fatalf("Failed to get a preferred client: %v", err)
	}

	chainID, err := preferredClient.ChainID(context.Background())
	if err != nil {
		log.Fatalf("Failed to get chain ID: %v", err)
	}
	fmt.Printf("Chain ID: %s\n", chainID.String())
}
```

---

### Advanced Configuration

```go
clientConfig := &clientmanager.ClientConfig{
	MonitorHealthInterval:   250 * time.Millisecond,
	HealthCheckRPCTimeout:   5 * time.Second,
	WaitHealthyPollInterval: 500 * time.Millisecond,
	MaxConcurrentETHCalls:   32,
}

managerConfig := &clientmanager.ClientManagerConfig{
	ClientConfig: clientConfig,
	PreferredClientMaxLatencyMultiplier: 3,
}
```

---

## Configuration Reference

### `ClientConfig` (Per-Client Settings)

| Field                     | Type            | Default | Description                                      |
| ------------------------- | --------------- | ------- | ------------------------------------------------ |
| `MonitorHealthInterval`   | `time.Duration` | `15s`   | How often to check node health.                  |
| `HealthCheckRPCTimeout`   | `time.Duration` | `5s`    | Timeout for a single health check RPC call.      |
| `WaitHealthyPollInterval` | `time.Duration` | `1s`    | How frequently to poll for health in wait loops. |
| `MaxConcurrentETHCalls`   | `int`           | `10`    | Maximum simultaneous RPC calls to this client.   |
| `Logger`                  | `Logger`        | default | Optional logger for client output.               |

### `ClientManagerConfig` (Manager-Level Settings)

| Field                                 | Type            | Default | Description                                                                                                         |
| ------------------------------------- | --------------- | ------- | ------------------------------------------------------------------------------------------------------------------- |
| `ClientConfig`                        | `*ClientConfig` | `nil`   | Configuration applied to all managed clients.                                                                       |
| `PreferredClientMaxLatencyMultiplier` | `int`           | `3`     | Multiplier for preferred client selection. Any healthy client with latency ≤ `minLatency * multiplier` is eligible. |
| `Logger`                              | `Logger`        | default | Optional logger for manager-level events and errors.                                                                |

---

## Contributing

We welcome contributions! To get started:

1. Fork the repository.
2. Create a new branch (`git checkout -b feature/your-feature`).
3. Commit your changes (`git commit -m 'Add new feature'`).
4. Push to your fork (`git push origin feature/your-feature`).
5. Open a pull request.

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
