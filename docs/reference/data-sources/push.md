# Push source

**Warning**: This is an _experimental_ feature. It's intended for early testing and feedback, and could change without warnings in future releases.

## Description

Push sources allow feature values to be pushed to the online store in real time. This allows fresh feature values to be made available to applications. Push sources supercede the 
[FeatureStore.write_to_online_store](https://rtd.feast.dev/en/latest/index.html#feast.feature_store.FeatureStore.write_to_online_store).

Push sources can be used by multiple feature views. When data is pushed to a push source, Feast propagates the feature values to all the consuming feature views.

Push sources must have a batch source specified. The batch source will be used for retrieving historical features. Thus users are also responsible for pushing data to a batch data source such as a data warehouse table. Feast plans on shipping `FeatureStore.write_to_offline_store` functionality soon, so users will be able to write data to the offline store just as easily as to the online store. When using a push source as a stream source in the definition of a feature view, a batch source doesn't need to be specified in the feature view definition explicitly.

## Stream sources
Streaming data sources are important sources of feature values. A typical setup with streaming data looks like:

1. Raw events come in (stream 1)
2. Streaming transformations applied (e.g. generating features like `last_N_purchased_categories`) (stream 2)
3. Write stream 2 values to an offline store as a historical log for training (optional)
4. Write stream 2 values to an online store for low latency feature serving
5. Periodically materialize feature values from the offline store into the online store for decreased training-serving skew and improved model performance

Feast now allows users to push features previously registered in a feature view to the online store for fresher features.

## Example
### Defining a push source
Note that the push schema needs to also include the entity

```python
from feast import PushSource, ValueType, BigQuerySource, FeatureView, Feature, Field
from feast.types import Int64

push_source = PushSource(
    name="push_source",
    batch_source=BigQuerySource(table="test.test"),
)

fv = FeatureView(
    name="feature view",
    entities=["user_id"],
    schema=[Field(name="life_time_value", dtype=Int64)],
    source=push_source,
)
```

### Pushing data
```python
from feast import FeatureStore
import pandas as pd

fs = FeatureStore(...)
feature_data_frame = pd.DataFrame()
fs.push("push_source_name", feature_data_frame)
```

See also [Python feature server](../feature-servers/python-feature-server.md) for instructions on how to push data to a deployed feature server. 

