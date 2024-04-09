# TODOs
- check front-end \[FE\] (Next.js + backend Python AWS Lambda functions interoperability to better understand how Next.js FE can repeatedly talk to the pipeline designed)
- check Facebook faiss + Python binding performance to assess similarity algorithm requirements (128 float numbers doesn't seem to be a lot yet it is better to check)
- decide what and how to test
  - AWS Lambda functions and their possible web controller replacements can and should be tested by mocks and integration tests
  - the whole pipeline should be tested for responsiveness/performance
- think of observability (CloudWatch + Managed Elastic and Graphana, externald DataDog, Newrelic etc.?)
  

# Questions about quality attributes

- expected load (number of queries for image understanding)
- possible/maximal image sizes (set requirements for uploading and processing and thus may affect responsiveness below)
- responsiveness expectations (hugely dependant on neural and similarity algorithms, may be easier to meet if giving out response in parts is acceptable, < 500 ms, < 1 sec, < 5 sec etc.)
- processed image retention (and thus caching, affects costs of storage)
- platforms (mobile, web-only, multiple platforms, other? influence front-end technology choice)


# Possible further development guesses

- Authentication and authorization (can be covered by adding a proxy doing the stuff)
- Caching (can be based on specialized image hashing algorithms, see Python imagehash lib and https://practicaldatascience.co.uk/data-science/how-to-use-image-hashing-to-identify-visually-similar-or-duplicate-images for an example)
  - one follow up question is performance of such image hashing algorithms (should be times faster than main image understanding algorithms to be practical)
- Adding other image understanding/processing features
- Scaling out (to larger number of queries -> may imply moving from lambdas to AWS ECS/EC2 or other compute resources to be kept within budget)
- Migrating to a cloud/technology of customer's preference


# Testing


# Frontend

- React
- React Native (if covering mobile as well
- Next.js + Capacitor (the latter if covering mobile as well)

https://devdactic.com/nextjs-and-capacitor (the previous article is on React + Capacitor)


# Next.js frontend + backend

Given neural network algorithms developed by a separate team with yet unclear compute load
it makes sense to go for custom backend instead of anything JavaScript/TypeScript bound.<br>

One more consideration is access to AWS resources like S3 which is more natural from an
AWS technology and not from a derivative Next.js frontend + backend deployment with harder
internal resource management.

# AWS Lambda functions

## Securing access
While authentication and authorization are out of scope for now it makes sense
to consider securing AWS Lambda functions accessed externally.
Recommended way is using API Gateway and JWTs
See https://docs.aws.amazon.com/lambda/latest/operatorguide/public-endpoints.html#api-endpoints

## Choosing compute resources for neural networks (image understanding)
Consider these AWS Lambda functions quotas https://docs.aws.amazon.com/lambda/latest/operatorguide/quotas.html
Note that concurrent executions number can be increased from default 1000/s to, let's say, 10000/s matching default of API Gateway.

# Assessing similarity algorithm and faiss in particular
