#On frontend

- React
- React Native (if covering mobile as well
- Next.js + Capacitor (the latter if covering mobile as well)

https://devdactic.com/nextjs-and-capacitor (the previous article is on React + Capacitor)

#On AWS Lambda functions

##Securing access
While authentication and authorization are out of scope for now it makes sense
to consider securing AWS Lambda functions accessed externally.
Recommended way is using API Gateway and JWTs
See https://docs.aws.amazon.com/lambda/latest/operatorguide/public-endpoints.html#api-endpoints

##Choosing as a compute resource for neural networks (image understanding)
Consider these quotas https://docs.aws.amazon.com/lambda/latest/operatorguide/quotas.html
