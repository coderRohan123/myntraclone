AWS GPU Deployment Options for Whisper.cpp Medium Model
Based on your requirements for deploying Whisper.cpp medium model on AWS with GPU acceleration for 50,000 monthly users processing 30-second audio clips, here are your optimal deployment options with detailed cost estimates.

Executive Summary
For your use case requiring pay-as-you-go pricing with support for 1,000 peak concurrent users and variable load (sometimes zero), AWS Batch with Spot instances offers the most cost-effective solution at approximately $1,506/month ($0.03 per request).

Monthly cost comparison showing AWS Batch with Spot instances as the most cost-effective option at $1,506/month, while auto-scaling solutions range from $2,607 to $4,681 per month
Monthly cost comparison showing AWS Batch with Spot instances as the most cost-effective option at $1,506/month, while auto-scaling solutions range from $2,607 to $4,681 per month
Deployment Architecture Options
🏆 Most Cost-Effective: AWS Batch + Spot Instances
Monthly Cost: ~$1,506 | Cost per request: $0.0301

Instance Type: G4dn.xlarge (T4 GPU, 4 vCPUs, 16GB RAM)

Spot Pricing: ~$0.21/hour (60% discount from on-demand)

Auto-scaling: Based on SQS queue depth

Architecture: SQS → AWS Batch → G4dn Spot instances

Pros: Lowest cost, excellent for variable workloads

Cons: Potential 2-minute interruption notices with Spot

🚀 Best Managed Solution: Auto-scaling G4dn.2xlarge on ECS
Monthly Cost: ~$2,607 | Cost per request: $0.0521

Instance Type: G4dn.2xlarge (2x T4 GPUs, 8 vCPUs, 32GB RAM)

Pricing: $0.752/hour with 15% average utilization

Auto-scaling: Built-in ECS service auto-scaling

Architecture: ALB → ECS Service → Auto Scaling Group

Pros: Good balance of cost and simplicity, reliable

Cons: Higher cost than Spot instances

🛡️ Enterprise/Fully Managed: SageMaker Async Inference
Monthly Cost: ~$3,509 | Cost per request: $0.0702

Instance Type: ml.g4dn.xlarge

Pricing: $0.50/hour with Savings Plan (32% discount)

Features: Built-in autoscaling, monitoring, queue management

Architecture: API Gateway → SageMaker Async Endpoint

Pros: Fully managed, scale-to-zero capability, enterprise features

Cons: 40% premium over raw EC2

Recent AWS GPU Pricing Updates
AWS recently announced significant GPU price reductions in June 2025:

P4d (A100): 33% price reduction

P5 (H100): Up to 45% price reduction

P5en (H200): 25% price reduction

These reductions make GPU instances more accessible, though G4dn instances (T4 GPUs) remain the most cost-effective for Whisper inference workloads.

Performance Considerations
Whisper.cpp Medium Model Performance
Processing Time: ~2.5 seconds for 30-second audio on T4 GPU

Concurrent Requests: T4 can handle 4-6 simultaneous transcriptions

Throughput: ~1.6 requests/second per T4 GPU

Memory Usage: Medium model requires ~2GB VRAM

Scaling Requirements
For your peak load of 1,000 concurrent users:

Target Response Time: 10 seconds (including queue time)

Instances Needed: ~63 G4dn.xlarge instances at peak[calculation]

Average Utilization: ~15% due to spiky workload patterns

Processing Capacity: 416.7 hours of audio per month

Alternative Options Not Recommended
AWS Lambda
❌ No GPU support available as of 2025

❌ Would require CPU-only inference (much slower)

❌ 15-minute execution timeout limit

AWS Fargate
❌ No GPU support available

❌ CPU-only performance inadequate for real-time requirements

❌ Higher cost than EC2 for sustained workloads

Implementation Recommendations
For Cost Optimization (AWS Batch + Spot)
Use SQS for request queuing and durability

Implement CloudWatch metrics for auto-scaling based on queue depth

Use whisper.cpp with CUDA for optimal T4 performance

Deploy in us-east-1 for lowest pricing

Implement graceful handling of Spot interruptions

For Reliability (ECS Auto-scaling)
Use Application Load Balancer for request distribution

Configure ECS Service Auto Scaling with target tracking

Implement health checks and circuit breakers

Use ECS Exec for debugging and monitoring

For Enterprise (SageMaker)
Leverage SageMaker Async Inference for built-in queue management

Use CloudWatch for comprehensive monitoring

Implement SageMaker Model Registry for model versioning

Configure auto-scaling policies for cost optimization

Additional Cost Considerations
Beyond compute costs, factor in:

Data Transfer: ~$4.39/month for audio uploads[calculation]

CloudWatch: ~$20/month for logs and metrics[calculation]

Load Balancer: ~$18/month (if using ALB)[calculation]

Storage: S3 costs for audio files (existing in your case)

Conclusion
AWS Batch with Spot instances provides the optimal balance of cost-effectiveness and performance for your variable workload pattern. At $1,506/month for 50,000 requests, it offers 70% cost savings compared to always-on solutions while maintaining the GPU performance necessary for real-time Whisper.cpp inference.

For production environments requiring higher reliability guarantees, the G4dn.2xlarge auto-scaling option at $2,607/month provides excellent value with built-in redundancy and simplified management.
