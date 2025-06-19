# OllyMan

OllyMan (ObservabilityManager) is a set of tools for easily bootstrapping, customizing, 
and scaling observability solutions. It provides clear definitions for the different components
of an observability stack, as well as interfaces for these definitions. 

## Purpose
OllyMan is an OpenTelemetry adn Kubernetes native observability stack manager. It will help you integrate
with OpenTelemetry quickly, and it will help you migrate to a new storage backend as you gain expertise
and a more intimate understanding out of what you need with your data.

It is **not** the mechanism by which you collect your data. For Kubernetes applications, my recommendation
is to use the otel-kube-stack to collect data from kubernetes. OllyMan provides the ability to install this for you,
with a reasonable default configuration. It is entirely reasonble to ignore this feature of OllyMan and install it
yourself if you wish, or already have it.

## Components

## Component Definitions

### Ingress Gateway*
This layer acts as the **entry point** for raw data, handling load balancing and context-aware data calculations. It is designed to **minimize processing** at 
this stage, receiving data in its most "raw" form (before filtering, enrichment, or re-attribution).  

- **Data Flow**:  
  - Receives **multiple data formats** (e.g., OTLP, Syslog, Prometheus, Influx, etc).  
  - Exports data as **OTLP (OpenTelemetry Line Protocol)**.  
  - **Optional Intermediate**: Kafka can be introduced here to act as a buffer between the **Ingress Gateway** and the **Data Processing** layer, enabling 
scalability and flexibility for future transitions to alternative **querying and storage** backends.  

- **Key Note**: The Ingress Gateway is intentionally simple, focusing on routing rather than processing.  

---

### Data Retention (Optional)
This is an **optional layer** that stores raw OTLP data. Its primary purpose is to facilitate migrations to new **querying 
and storage** backends by allowing replay of data from this layer, and it secondarily provides the ability to buffer data
here before sending it to the **data processing** layer.

- **Tradeoff**:  
  - **Storage Overhead**: Storing raw OTLP data is resource-intensive.  
  - **Use Case**: Unclear if needed, but provides flexibility in the next layer. 

---

### Data Processing
This is the **core processing layer**, designed to handle high-volume data with load balancing and advanced data manipulation.  

- **Key Features**:  
  - **Load-Balanced OpenTelemetry Collectors**: Ensures scalability and fault tolerance.  
  - **Data Manipulation**: Filtering, re-attribution, enrichment, and transformation of data occur here.  
  - **Data Flow**: Receives OTLP from the **Ingress Gateway** and exports OTLP to the next layer.  

- **Design Rationale**: This layer is the "engine" of the stack, ensuring data is processed efficiently before being stored or queried.  

---

### Querying and Storage
This layer provides the **plug-and-play observability backend** and enforces **OTLP** as the data format.  

- **Key Design**:  
  - A **single OpenTelemetry Collector** acts as a bridge between the **Data Processing** layer and this layer.  
  - This collector is **exclusively responsible for transforming** data into the format required by the **querying and storage** backend (e.g., a time-series 
database or analytics platform).  
  - **Data Flow**: Receives OTLP from the Data Processing layer and outputs processed data to the backend.  

- **Flexibility**: The use of OTLP ensures compatibility with a wide range of tools, making this layer easy to replace or extend.  

#### Options

1. ClickHouse
2. Grafana LGTM
3. ?
4. Bring-your-own-backend
    4a. This means you would just be using OllyMan as a way to scale your telemetry ingestion and forward it to its final destination

---

