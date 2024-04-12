# TODOs
- ~~check front-end \[FE\] (Next.js + backend \[BE\]) to Python AWS Lambda functions interoperability to better understand how Next.js FE can repeatedly talk to the pipeline designed~~
  - => interoperability is OK
    - FE UI can be repeatedly update by a set of changes of the page (React) states
    - auth can be put to Next.js BE server actions except for uploading via presigned URLs maybe
- ~~check Facebook faiss + Python binding performance to assess similarity algorithm requirements (128 float numbers doesn't seem to be a lot yet it is better to check)~~
  - theory:
    - GPU based algorithms may be even faster than 3rd - NOT TESTED having no GPU
  - practice on my i5-10310U (mobile, low-energy) quad core HT CPU (8 vCPU) and 32GB RAM:
    - building (quantizing, training, adding) indexes and search consumed all 8 vCPUs for 100%
    - Voronoi-cells-based (2nd) algorithm searches a few times faster than L2 (1st) <br>
    tested for 1-10M 128*float32 DB and 1-10000 vectors looked up <br>
    yet it takes about ten times more time to build an index (10M -> 1 sec vs 10 sec)
      - 1st/2nd RAM for 10M is ~10 GB
    - the 3rd algorithm searches ~10x faster than 1st & about ~5x faster than 2nd
      - yet it builds its index ~10x slower than 2nd and ~50x slower than 1st
      - RAM for 10M is ~6 GB
    - faiss based similarity algorithm requires a decent machine in case of <br>
    lots of search requests let's say (10K in a second)
      - a good machine (hundreds CPUs / 4-8 powerful GPUs) can cost hundreds/thousands USD a month in AWS EC2
    - 10M+ vectors in index and acceptable delays of minutes may allow persisting the whole index build
      - and add something to the index only once in a while
      - restoring 10-50 GB may take 10 sec to 1 minute <br>
      (not considering RAID, Luster and other faster filesystem approaches)
    - faiss offers an option to search not in RAM but on disk
- decide what and how to test
  - AWS Lambda functions and their possible web controller replacements can and should be tested by mocks and integration tests
  - the whole pipeline should be tested for responsiveness/performance
  - Next.js (other JS) FE can be checked for absence of any corporate/cloud credentials
  - faiss based similarity algorithm can be tested:
    - volume/performance tests
    - integration tests with DBs and with batching mechanism (if needed by picture DB volume/rate)
- think of observability (CloudWatch + Managed Elastic and Graphana, externald DataDog, Newrelic etc.?)
- think of deployment (possibly Continous Deployment)
- ~~decide on where to orchestrate flow - Next.js/Vercel/3rd party BE vs cloud provider triggers, lambdas & other~~
  - ~~in cloud is more reliable and may be a requested feature of processing even when Next.js/Vercel/3rd party FE/BE fails~~
    - can be built on S3 triggers + lambdas
    - having about 4-5 lambdas and 100 requests per 1 sec may results in just about $200 a month for AWS Lambda costs (1B lambda function calls)
      - if functions take less than 3 sec and <500 MB RAM on average
    - CONS: <br>
      - anyway will require about the same number of requests from N
  - orchestration on Next.js/Vercel/3rd party BE seems easier to implement <br>
  (no need in additional in-cloud tools and mechanism to be coded, configured, debugged and tested)
    - can be chosen first to get MVP (minimal viable product) faster <br>
    and can be relatively easy migrated to cloud tools later if the need arise


# Questions about quality attributes

- expected load (number of queries for image understanding)
- possible/maximal image sizes (set requirements for uploading and processing and thus may affect responsiveness below)
  - e.g. API Gateway proxy S3 access can be used if image size is no more than 10MB
- responsiveness expectations (hugely dependant on neural and similarity algorithms, may be easier to meet if giving out response in parts is acceptable, < 500 ms, < 1 sec, < 5 sec etc.)
  - e.g. API Gateway proxy S3 access may be used instead of presigned URLs to make only one http request instead of two
    - unlikely to be a problem in practice if something like Vercel or AWS VPC is used to host/deploy to BE
- processed image retention (and thus caching, affects costs of storage)
- platforms (mobile, web-only, multiple platforms, other? influence front-end technology choice)
- image/vectors DB to search size (100K-100M ?)


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

See appropriate TODO item that is strikethrough.
