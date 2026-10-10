Create a production-ready static website deployed on:





S3 (for hosting)



CloudFront (for CDN + HTTPS)



Route53 (for the domain)

This is a classic AWS DevOps workflow.



Tasks

1. Static Website Bucket





Create an S3 bucket



Enable Static Website Hosting



Upload a simple index.html and error.html



Make bucket objects publicly readable (bucket policy)



2. Set Up CloudFront





Origin: S3 website endpoint



Enable:





HTTPS



Compress objects



Cache policy (Managed – CachingOptimized)



Default behaviour: allow GET/HEAD



Add an ACM certificate for your chosen domain (if using Route53) (OPTIONAL)



3. Route53 Setup





Create a hosted zone (if not already)



Add an A record (Alias) > point it to CloudFront distribution



Confirm it resolves to the CDN endpoint



4. Testing





Visit your domain



Confirm CloudFront is serving content



Confirm caching:





Edit your index.html



Invalidate CloudFront cache



Refresh page



Bonus (Optional)





Add a CI/CD pipeline using GitHub Actions → automatic deploy to S3



Add security headers through CloudFront functions



Add Lambda@Edge to rewrite URLs




