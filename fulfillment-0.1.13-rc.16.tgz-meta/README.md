# fulfillment

`fulfillment` is a service for fulfilling marketplace orders.

## Amazon Web Services

To transact via AWS:

- A customer purchases Giant Swarm via the AWS Marketplace.
- The customer is redirected to the `fulfillment` service webhook, with a token.
- The `fulfillment` service renders a webpage with a form (with the token embedded), so the customer can add their email address.
- When the form is submitted, the `fulfillment` service requests data from AWS about the customer's subscription using the token. This information is then sent to Slack with the customer's email, where operations then reaches out to the customer via the email.

See [here](https://docs.aws.amazon.com/marketplace/latest/userguide/saas-products.html) for AWS documentation. [This page](https://docs.aws.amazon.com/marketplace/latest/userguide/saas-product-customer-setup.html) specifically details the process on how a customer fulfills via the AWS Marketplace.
