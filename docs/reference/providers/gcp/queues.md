---
title: Queues
description: How Nitric deploys Queues to Google Cloud
---

Nitric queues are deployed using [pubsub](https://cloud.google.com/pubsub) topics with a single pull subscription when deploying to GCP.

## GCP Resources

The following resources are created when deploying queues to GCP:

- Pubsub topic
- Pubsub topic subscription (pull)

## Deployment

During deployment, your queues will be provisioned as follows:

- The queue is deployed as a pubsub topic.
- The queue will have a single pull subscription created for it.