## CloudWatch Dashboards: Infrastructure vs. Application Monitoring

For effective monitoring, it's best practice to create separate CloudWatch dashboards for infrastructure and application monitoring:

- **Infrastructure Monitoring:**  
  AWS services such as EC2, EBS, and EFS automatically collect and publish infrastructure metrics to CloudWatch. No manual intervention is required for these metrics.

- **Application Monitoring:**  
  Application-level metrics need to be manually instrumented by the developer and published as custom metrics to CloudWatch. Additionally, application logs must be explicitly configured to be sent to CloudWatch Logs.

This separation ensures clear visibility and management of both system health and application performance.
