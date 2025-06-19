# OllyMan

OllyMan (ObservabilityManager) is a set of tools for easily bootstrapping, customizing, 
and scaling observability solutions. It provides clear definitions for the different components
of an observability stack, as well as interfaces for these definitions. 


## Definitions

### Ingress Gateway
The ingress gateway serves as the single-point-of-entry to the rest of the stack. It largely only serves
the purpose of load balancing the data-processing section, which is required to allow for context-aware
data calculations that happen in the **data processing** layer. This is a simple layer that is designed
to do as little work as possible. This layer has data in its most "raw" form, prior to any filtering, 
enrichment, or other alterations that happen in data processing. For that reason, it is worth noting that
one might consider introducing kafka as a medium between this layer and the data processing layer. This is
entirely optional, but would allow for a more seamless transistion to an alternative **querying and storage** 
layer if one were to choose to migrate later on.

This layer receives **multiple** data formats, and exports it as OTLP.

### Data Retention **optional**
This is an optional layer that allows for preserving raw otlp data. This primarily serves the purpose of 
allowing easier transitions to new **querying and storage** backends, since data can be replayed from this
point into the new backend. Naturally, there is a substantial storage overhead to storing raw data at this
point.

### Data Processing
This layer is what allows the stack to process large quantities of data. This is a load balanced set of 
OpenTelemetry Collectors, and it is also the layer where all filtering, processing, re-attribution, and enrichment
of data occurs. 

This layer recieves OTLP, and exports OTLP.

### Querying and Storage
This layer is what provides the "plug and play" observability backend. We enforce OTLP (OpenTelemetry Line Protocol)
as the data format, which is made possible by putting a single collector at the front of the **querying and storage**
layer, which is used exclusively for the purpose of transforming data into whatever format the **querying and storage**
layer requires. 
