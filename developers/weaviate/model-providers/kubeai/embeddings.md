---
title: Text Embeddings
sidebar_position: 20
# image: og/docs/integrations/provider_integrations_openai.jpg
# tags: ['model providers', 'openai', 'embeddings']
---

# KubeAI Embeddings with Weaviate

import BetaPageNote from '../_includes/beta_pages.md';

<BetaPageNote />

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import FilteredTextBlock from '@site/src/components/Documentation/FilteredTextBlock';
import PyConnect from '!!raw-loader!../_includes/provider.connect.py';
import TSConnect from '!!raw-loader!../_includes/provider.connect.ts';
import PyCode from '!!raw-loader!../_includes/provider.vectorizer.py';
import TSCode from '!!raw-loader!../_includes/provider.vectorizer.ts';

Weaviate's integration with OpenAI's APIs allows you to access KubeAI models' directly from Weaviate.

[KubeAI](https://github.com/substratusai/kubeai) provides private OpenAI compatible API endpoint for OSS or custom LLMs for embeddings.

[Configure a Weaviate vector index](#configure-the-vectorizer) to use an OpenAI embedding model, and Weaviate will generate embeddings for various operations using the specified model. This feature is called the *vectorizer*.

At [import time](#data-import), Weaviate generates text object embeddings and saves them into the index. For [vector](#vector-near-text-search) and [hybrid](#hybrid-search) search operations, Weaviate converts text queries into embeddings.

![Embedding integration illustration](../_includes/integration_openai_embedding.png)

## Requirements

### Weaviate configuration

Your Weaviate instance must be configured with the OpenAI vectorizer integration (`text2vec-openai`) module.

<details>
  <summary>For Weaviate Cloud (WCD) users</summary>

This integration is enabled by default on Weaviate Cloud (WCD) serverless instances.

</details>

<details>
  <summary>For self-hosted users</summary>

- Check the [cluster metadata](../../config-refs/meta.md) to verify if the module is enabled.
- Follow the [how-to configure modules](../../configuration/modules.md) guide to enable the module in Weaviate.

</details>

### API credentials

You must provide a valid OpenAI API key to Weaviate for this integration. However, KubeAI ignores the OpenAI API key. So you can provide any value for the API key.

Provide the API key to Weaviate using one of the following methods:

- Set the following environment variable `export OPENAI_APIKEY=thisIsIgnored`.
- Provide the API key at runtime, as shown in the examples below.

<Tabs groupId="languages">

 <TabItem value="py" label="Python API v4">
    <FilteredTextBlock
      text={PyConnect}
      startMarker="# START OpenAIInstantiation"
      endMarker="# END OpenAIInstantiation"
      language="py"
    />
  </TabItem>

 <TabItem value="js" label="JS/TS API v3">
    <FilteredTextBlock
      text={TSConnect}
      startMarker="// START OpenAIInstantiation"
      endMarker="// END OpenAIInstantiation"
      language="ts"
    />
  </TabItem>

</Tabs>

## Configure the vectorizer

[Configure a Weaviate index](../../manage-data/collections.mdx#specify-a-vectorizer) to use an KubeAI embedding model by setting the vectorizer as follows:

You need to specify a model name for it work with KubeAI. No default model is
configured.

KubeAI comes with a `nomic-embed-text-cpu` model that can be used for text embeddings.
You can enable the model by setting `enabled: true` in the `kubeai-values.yaml` file.
Note that the model name is `text-embedding-ada-002`.

Create a file named `kubeai-values.yaml` with the following content:
```yaml
models:
  catalog:
    text-embedding-ada-002:
      enabled: true
      minReplicas: 1
      features: ["TextEmbedding"]
      owner: nomic
      url: "ollama://nomic-embed-text"
      engine: OLlama
      resourceProfile: cpu:1
```

Afterwards apply the new configuration to the KubeAI Helm chart:
```bash
helm repo add kubeai https://www.kubeai.org
helm repo update
helm upgrade --install kubeai kubeai/kubeai \
    -f ./kubeai-values.yaml --reuse-values
```

Now you should be able to configure the vectorizer with the model name `text-embedding-ada-002`.

<Tabs groupId="languages">
  <TabItem value="py" label="Python API v4">
    <FilteredTextBlock
      text={PyCode}
      startMarker="# START FullVectorizerKubeAI"
      endMarker="# END FullVectorizerKubeAI"
      language="py"
    />
  </TabItem>

  <TabItem value="js" label="JS/TS API v3">
    <FilteredTextBlock
      text={TSCode}
      startMarker="// START FullVectorizerKubeAI"
      endMarker="// END FullVectorizerKubeAI"
      language="ts"
    />
  </TabItem>

</Tabs>

## Data import

After configuring the vectorizer, [import data](../../manage-data/import.mdx) into Weaviate. Weaviate generates embeddings for text objects using the specified model.

<Tabs groupId="languages">

 <TabItem value="py" label="Python API v4">
    <FilteredTextBlock
      text={PyCode}
      startMarker="# START BatchImportExample"
      endMarker="# END BatchImportExample"
      language="py"
    />
  </TabItem>

 <TabItem value="js" label="JS/TS API v3">
    <FilteredTextBlock
      text={TSCode}
      startMarker="// START BatchImportExample"
      endMarker="// END BatchImportExample"
      language="ts"
    />
  </TabItem>

</Tabs>

:::tip Re-use existing vectors
If you already have a compatible model vector available, you can provide it directly to Weaviate. This can be useful if you have already generated embeddings using the same model and want to use them in Weaviate, such as when migrating data from another system.
:::

## Searches

Once the vectorizer is configured, Weaviate will perform vector and hybrid search operations using the specified KubeAI model.

![Embedding integration at search illustration](../_includes/integration_openai_embedding_search.png)

### Vector (near text) search

When you perform a [vector search](../../search/similarity.md#search-with-text), Weaviate converts the text query into an embedding using the specified model and returns the most similar objects from the database.

The query below returns the `n` most similar objects from the database, set by `limit`.

<Tabs groupId="languages">

 <TabItem value="py" label="Python API v4">
    <FilteredTextBlock
      text={PyCode}
      startMarker="# START NearTextExample"
      endMarker="# END NearTextExample"
      language="py"
    />
  </TabItem>

 <TabItem value="js" label="JS/TS API v3">
    <FilteredTextBlock
      text={TSCode}
      startMarker="// START NearTextExample"
      endMarker="// END NearTextExample"
      language="ts"
    />
  </TabItem>

</Tabs>

### Hybrid search

:::info What is a hybrid search?
A hybrid search performs a vector search and a keyword (BM25) search, before [combining the results](../../search/hybrid.md#change-the-ranking-method) to return the best matching objects from the database.
:::

When you perform a [hybrid search](../../search/hybrid.md), Weaviate converts the text query into an embedding using the specified model and returns the best scoring objects from the database.

The query below returns the `n` best scoring objects from the database, set by `limit`.

<Tabs groupId="languages">

 <TabItem value="py" label="Python API v4">
    <FilteredTextBlock
      text={PyCode}
      startMarker="# START HybridExample"
      endMarker="# END HybridExample"
      language="py"
    />
  </TabItem>

 <TabItem value="js" label="JS/TS API v3">
    <FilteredTextBlock
      text={TSCode}
      startMarker="// START HybridExample"
      endMarker="// END HybridExample"
      language="ts"
    />
  </TabItem>

</Tabs>

## References

### Vectorizer parameters

- `model`: The KubeAI model name.
- `dimensions`: The number of dimensions for the model.
- `baseURL`: The OpenAI compatible endpoint provided by KubeAI.

In most cases the `baseURL` is `http://kubeai/openai`. Unless you have Weaviate
deployed in a different cluster or namespace.

## Further resources

### Other integrations

- [KubeAI generative models + Weaviate](./generative.md).

### Code examples

Once the integrations are configured at the collection, the data management and search operations in Weaviate work identically to any other collection. See the following model-agnostic examples:

- The [how-to: manage data](../../manage-data/index.md) guides show how to perform data operations (i.e. create, update, delete).
- The [how-to: search](../../search/index.md) guides show how to perform search operations (i.e. vector, keyword, hybrid) as well as retrieval augmented generation.

### External resources

- OpenAI [Embed API documentation](https://platform.openai.com/docs/api-reference/embeddings)

## Questions and feedback

import DocsFeedback from '/_includes/docs-feedback.mdx';

<DocsFeedback/>