# Image understanding application design

## Frontend architecture

Assuming responsiveness of the application is important the application
frontend is React.js that updates its UI gradually with results got in chunks over time after an upload is made.
UI updates are done via states.
The React.js application is deployed to S3 and delivered to users via AWS
CloudFrond (CDN).

### Communication with backend

Assuming image size can be more than 10 MB and images should be processed
pristine React.js frontend uploads an image to S3 storage via AWS API Gateway
and Lambda function generating a presigned URL working only for HTTP PUT method
and the frontend origin.
All presigned URLs expire in seconds and are used by React.js frontend for
direct uploads of input to and downloads of results from S3.
React.js frontend mantains a WebSocket (Socket.io) connection to and is
notified when intermediate and final results of image understanding are
available by API Gateway WebSocket API and its backing AWS Lambda functions.
When results are available React.js frontend downloads them using API Gateway,
Lambda functions and presigned URLs and updates its own state and UI.

## Backend architecture

Assuming Algorithms A and B take not milliseconds but seconds to complete
processing their respective steps of image understanding compute resources are
ECS containers and backing EC2 instances. In case of special requirements
compute resources can be pure EC2.
All algorithms take input from and put results to S3 storage.
Assuming image database is 10+ million images Approximate Nearest Neighbor is a
faiss based algorithm and uses ECS compute resources.
Each ECS container of the algorithm builds its index reading data from an AWS
RDS PostgreSQL instance.

The flow of image understanding starts from an image upload to an S3 bucket and
proceeds by S3 triggers and AWS Lambda functions invoking algorithms and
notifying React.js backend via AWS API Gateway WebSocket API of completion of
stages.
All inputs and results are stored using S3.
The above is how AI algorithm parts are integrated into the image understanding
application.

Proposed language for all Lambda functions is Python yet TypeScript can be used
as an alternative to make both frontend and backend written in one language.

Simple caching of results for inputs using hashing is implemented to avoid
processing of the same input multiple times. Hashing and verifying existence of
previous results is responsibility of Lambda functions of API Gateway.
Image understanding results retention is set and expires at can be renewed on
new fetches per corporate policies.

128 floating point numbers vector part of image database compiled by the client
is hosted in an Amazon RDS PostgreSQL database.
The very images are stored using S3.
Mapping of vectors to images is a table of the database.

## Cloud deployment

Cloud deployment is done by Terraform.
All the AWS objects are managed by Terraform configuration.
React.js frontend is packed as a CloudFront distribution.
Algorithms are packed as Docker images unless the developers set other
requirements for algorithms A and B.

Image database, both vectors and images, is initially populated and later
extended by automation resembling a modified version of the main use case flow
with disabled UI, algorithm A and similarity search and enabled insertion of
vectors and images to both PostgreSQL and S3 database parts.
The automation is adapated to be used by scripts.
If a requirement is set the main use case can also be modified to further extend the database. In such a case database image crop files are saved and do not expire when stored in an S3 bucket.

Each lambda layer needed by Lambda functions is packed by another automation piece resembling a series of package installations to an environment and packing the result of installations according to Lambda layer packing guidelines published by Amazon AWS. Packed lambda layers are managed by Terraform configuration.

## Testing

All tests below are run before full deployment to production.

Unit/mock testing of code of Lambda functions test input validation and post
conditions.
Such tests run on each check-in to monitored source control branches failing
the build if not passed.
Depending on source code hosting GitHub, GitLab actions or separate AWS compute
resource is used to run such tests.

Apart from unit/mock tests of code of Lambda functions the image understanding
application is tested with integration, performance, volume and security tests.

Frontend integration tests are a Selenium test suite making sure that React.js frontend successfully updates its state gradually while interacting with a simple mock backend via both HTTP(S) and WebSocket and getting mock results.
Backend integration tests deploy subsets of cloud infrastructure and test their
interaction with their counterparts or mock where applicable.
Examples are tests making sure:
- Terraform configured S3 triggers trigger on certain S3 event like an image file upload
- a Lambda layer / set of layers provides necessary dependencies to make certain
lambda work
- a Lambda function is properly notifies a mock frontend via API Gateway
WebSocket API.
Backend integration tests are meant to test the Terraform configuration and
Lambda functions code result in backend parts behaving as expected.

Security test examples are tests making sure that:
- presigned URLs do not provide wrong HTTP methods or path access and expire as
expected
- frontend facing Lambda functions cannot be misused to provide access or
functionality beyond expected (e.g. by consuming failure causing inputs)
- non frontend facing endpoints are not availabe from outside VPC
- API Gateway APIs does drop duly on wrong/forbidden incoming requests

Performance and volume test examples are tests of image similarity faiss based algorithm and the behaviour of the flow under high upload rates and large database volume.
Especially mportant tests are tests having both expected peak request rates and
maximum projected DB volumes.

In order to identify most important required tests a diagram or set of diagrams
of the application parts and their interactions is to be analyzed.

All code is checked with appropriate static code analysis tools and linters.
Such tools run on each check-in to monitored branches failing CI/CD builds in
case of severe issues defined by their rules.
Examples:
- Checkov for Terraform and Dockerfile code
- ESCheck for React.js frontend
- Python linters

Other test that run on each check-in to monitored branches are chosen depending
on their cost and time needed. Some others run regularly (e.g. overnight).
Most costly tests run manually and/or when code is critically responsible for
passing them is modified.