# DevOps Interview Questions for Senior Engineers

## Infrastructure as Code & Configuration Management

### 1. Explain Infrastructure as Code (IaC)
   **Answer:**
   - Infrastructure as Code treats infrastructure like software code
   - Declarative approach: Describe desired state, let tools achieve it
   - Benefits: Version control, reusability, automated testing, consistency

   **Tools & Technologies:**
   - **Terraform**: Declarative, multi-cloud support
   - **CloudFormation**: AWS-specific, native AWS integration
   - **Pulumi**: Imperative code with access to full languages (TypeScript, Python)

   **Best Practices:**
   - Remote state management with locking
   - Module-based architecture for reusability
   - CI/CD integration for infrastructure changes

### 2. How do you handle configuration drift?

   **Answer:**
   - Configuration drift: Deviation from intended configuration over time
   - Detection methods:
     - Scheduled scans/audits
     - Agent-based monitoring (Chef InSpec, Ansible Tower)
     - Golden image comparisons
   - Prevention strategies:
     - Immutable infrastructure
     - Automated remediation
     - Policy as Code (Open Policy Agent)

   **Tools:**
   - **Terraform drift detection**: `terraform plan`
   - **AWS Config**: Continuous compliance monitoring
   - **Drift remediation**: AWS Config Rules, Lambda functions

## Continuous Integration/Continuous Deployment

### 3. Design a CI/CD pipeline for microservices

   **Component Breakdown:**
   ```
   Source Code (Git) → Build → Test → Artifact → Deploy → Monitor
   ```

   **Stages:**
   1. **Source Stage**: Webhooks trigger on commits/PR merges
   2. **Build Stage**: Multi-stage Docker builds, security scanning
   3. **Test Stage**: Unit, integration, contract tests
   4. **Artifact Stage**: Push to registry (Docker Hub, ECR, GHCR)
   5. **Deploy Stage**: Progressive rollout (canary/blue-green)
   6. **Monitor Stage**: Synthetic tests, probe deployments

   **Advanced Patterns:**
   - **Trunk-based development**: Short-lived feature branches
   - **GitOps**: Infrastructure as code with Git operations
   - **Progressive delivery**: Feature flags, canary deployments

### 4. How do you handle database schema changes in CI/CD?

   **Strategies:**
   - **Forward-only migrations**: No rollbacks in production
   - **Blue-green deployments**: Switch between identical environments
   - **Multi-stage rollouts**: Deploy schema changes before code

   **Tools:**
   - **Flyway**: Version-based SQL migrations
   - **Liquibase**: XML/directory-based migrations
   - **Alembic**: Python-based database migrations

   **Safety Measures:**
   - Schema validation in CI
   - Lock mechanisms during migration
   - Backup verification before migration

## Container Orchestration & Kubernetes

### 5. Explain Kubernetes resource management

   **Resource Types:**
   - **Requests**: Guaranteed allocation for pods
   - **Limits**: Maximum allowed usage before throttling/OOM kill
   - **QoS Classes**:
     - Guaranteed: requests = limits
     - Burstable: requests < limits
     - BestEffort: no requests/limits

   **Resource Quotas & Limits:**
   - **ResourceQuota**: Namespace-level resource constraints
   - **LimitRange**: Default requests/limits for pods

   **Monitoring:**
   - Metrics Server for resource utilization
   - Prometheus custom metrics for autoscaling
   - Horizontal/Vertical Pod Autoscaler

### 6. How do you design for high availability in Kubernetes?

   **Cluster Level:**
   - Multi-master (control plane) with etcd clustering
   - Multi-AZ node distribution
   - Pod disruption budgets for maintenance

   **Application Level:**
   - ReplicaSets with anti-affinity rules
   - Readiness/liveness probes
   - Rolling update strategies

   **Network Level:**
   - LoadBalancer service type
   - Ingress controllers with redundancy
   - Service mesh (Istio) for circuit breaking

## Monitoring & Observability

### 7. Explain the four pillars of observability

   **1. Metrics:**
   - Quantitative measurements over time
   - USE method: Utilization, Saturation, Errors
   - RED method: Request rate, Error rate, Duration

   **2. Logs:**
   - Structured logging for debugging
   - Centralized logging (ELK, Loki)
   - Log aggregation and correlation

   **3. Traces:**
   - Distributed request tracking
   - Trace context propagation
   - Service dependency mapping

   **4. Events:**
   - Discrete occurrences (deploys, incidents)
   - Event streaming (Kafka, CloudWatch Events)
   - Event correlation and alerting

### 8. Design a monitoring strategy for cloud-native applications

   **Infrastructure Monitoring:**
   - System metrics (CPU, memory, disk, network)
   - CloudWatch, Prometheus + node_exporter
   - Synthetic monitoring for external services

   **Application Monitoring:**
   - Business metrics (user registration, revenue)
   - Performance metrics (latency, throughput, error rates)
   - Custom dashboards and SLOs

   **Log Aggregation:**
   - Structured JSON logging
   - Fluent Bit for log shipping
   - Elasticsearch/Loki for storage and querying

   **Alerting:**
   - Multi-level alerts (page, ticket, email)
   - Alert fatigue prevention
   - On-call rotation with escalation policies

## Site Reliability Engineering

### 9. Explain Service Level Indicators (SLIs), Objectives (SLOs), and Agreements (SLAs)

   **SLI (Service Level Indicator):**
   - Measurable aspect of service performance
   - Examples: Request latency < 100ms, Availability = 99.9%

   **SLO (Service Level Objective):**
   - Target value for an SLI over a time period
   - "Latency < 100ms for 99% of requests in rolling 28 days"

   **SLA (Service Level Agreement):**
   - Contractual commitment to SLOs
   - Includes consequences for violating SLOs

   **Error Budget:**
   - 100% - SLO percentage (e.g., 99.9% SLO = 0.1% error budget)
   - Spent on innovation when under budget
   - Triggers incident response when exceeded

### 10. How do you conduct a post-mortem analysis?

    **Structure:**
    **Timeline:** Chronological event sequence
    **Blame-free Culture:** Focus on systems, not people
    **Contributing Factors:** Root cause identification
    **Immediate Actions:** Quick fixes applied
    **Long-term Solutions:** Systemic improvements
    **Prevention Measures:** Process/policy changes

    **Documentation:**
    - Store in accessible location (wiki, Confluence)
    - Follow-up actions with owners and timelines
    - Lessons learned for knowledge sharing

## Cloud Architecture & Cost Optimization

### 11. Design cost-effective cloud architecture

    **Compute Cost Optimization:**
    - **Reserved Instances**: 1-3 year commitments (up to 75% savings)
    - **Spot Instances**: Interruptible instances (up to 90% savings)
    - **Auto-scaling**: Pay for what you need

    **Storage Cost Optimization:**
    - **EBS volume types**: gp3 > gp2 > io1 (cost vs performance)
    - **S3 storage classes**: Intelligent tiering, Glacier for cold data
    - **Data lifecycle policies**: Automated data movement

    **Network Cost Optimization:**
    - **Regional data transfer**: Minimize cross-region traffic
    - **CDN usage**: Global content delivery
    - **Gateway Load Balancer**: Centralized network services

    **Monitoring & Alerting:**
    - Cost allocation tags
    - Budget alerts and reports
    - Reserved Instance utilization tracking

### 12. Explain multi-cloud vs hybrid-cloud strategies

    **Multi-cloud Advantages:**
    - Avoid vendor lock-in, leverage best-of-breed services
    - Disaster recovery across providers
    - Cost optimization through competition
    - Geographic compliance (data residency)

    **Multi-cloud Challenges:**
    - Complexity in tooling and operations
    - Network peering costs
    - Skillset diversity requirements
    - Configuration consistency

    **Hybrid Cloud:**
    - Maintain legacy systems on-premises
    - Burst to cloud during peak loads
    - Data sovereignty/financial compliance
    - Gradual migration path

## Security & Compliance

### 13. Implement DevSecOps practices

    **Shift-left Security:**
    - Security testing in CI/CD pipeline
    - Infrastructure security scanning
    - Dependency vulnerability assessment

    **Security Tools Integration:**
    - **SAST**: SonarQube, ESLint security rules
    - **DAST**: OWASP ZAP, Burp Suite
    - **Container Security**: Clair, Trivy
    - **Secrets Management**: Vault, AWS Secrets Manager

    **Governance:**
    - Policy as Code (Open Policy Agent)
    - Compliance as Code with automated checks
    - Security incident response

### 14. How do you ensure infrastructure security?

    **Network Security:**
    - **Zero Trust**: Never trust, always verify
    - **Network Segmentation**: VPC/subnet isolation
    - **Security Groups/Network ACLs**: Traffic filtering

    **Access Management:**
    - **Principle of Least Privilege**: Minimal required permissions
    - **RBAC/ABAC**: Role/attribute-based access
    - **MFA**: Multi-factor authentication everywhere

    **Data Protection:**
    - **Encryption at Rest**: Database/AWS KMS
    - **Encryption in Transit**: TLS 1.3
    - **Data Loss Prevention**: Classification and monitoring

## Performance Engineering & Scalability

### 15. How do you troubleshoot performance issues in production?

    **Methodical Approach:**
    1. **Metrics Analysis**: CPU, memory, disk I/O, network
    2. **Log Review**: Error patterns, slow queries
    3. **APM Tools**: Application performance monitoring
    4. **Infrastructure Monitoring**: System-level bottlenecks

    **Common Performance Issues:**
    - **Database**: Missing indexes, N+1 queries, connection pools
    - **Application**: Memory leaks, garbage collection pauses
    - **Infrastructure**: Resource saturation, network bottlenecks
    - **External Services**: Third-party API throttling, DNS resolution

    **Diagnostic Tools:**
    - `strace`, `tcpdump` for low-level debugging
    - `perf`, `flamegraphs` for CPU profiling
    - Database slow query logs and explain plans

### 16. Explain auto-scaling for web applications

    **Horizontal Scaling:**
    - **Application Layer**: Increase pod/container instances
    - **Database Layer**: Read replicas, connection pooling
    - **Cache Layer**: Redis/Memcached clusters

    **Vertical Scaling:**
    - Increase instance size (CPU, memory)
    - Usually stopgap, not long-term solution

    **Scaling Triggers:**
    - **CPU Utilization**: >70% scale up, <30% scale down
    - **Request Queue Depth**: Too many pending requests
    - **Custom Metrics**: Business metrics (orders/minute)

    **Scaling Considerations:**
    - **Startup Time**: Fast-scaling containers vs slow VMs
    - **State Management**: Stateless applications scale easily
    - **Data Consistency**: Scaling stateful services
    - **Cost Monitoring**: Scale-down to control expenses

## Incident Response & Chaos Engineering

### 17. How do you prepare for production incidents?

    **Preparation Phase:**
    - **Runbooks**: Documented incident response procedures
    - **War Room**: Communication channels and tools
    - **Escalation Matrix**: Who to contact at what severity

    **During Incident:**
    - **Alert Triage**: Assess impact and urgency
    - **Communication**: Regular updates to stakeholders
    - **Containment**: Stop bleeding, isolate affected systems
    - **Root Cause Analysis**: While fixing, understand why

    **Post-Incident:**
    - **Blame-free Post-mortem**: Learning-focused analysis
    - **Improvement Implementation**: Fix root causes
    - **Documentation Updates**: Improve runbooks and monitors

### 18. Explain chaos engineering and game days

    **Principles:**
    - **Hypothesis-driven**: Form hypothesis about system behavior
    - **Minimize blast radius**: Start small, expand carefully
    - **Automate experiments**: Don't rely on manual intervention
    - **Run in production**: Test production configurations

    **Types of Experiments:**
    - **Network Chaos**: Network delay, packet loss, partitions
    - **Resource Chaos**: CPU/memory exhaustion
    - **Application Chaos**: Kill pods, inject exceptions
    - **Infrastructure Chaos**: AZ failures, instance termination

    **Tools:**
    - **Chaos Monkey**: Random instance termination
    - **Gremlin**: Commercial chaos engineering platform
    - **LitmusChaos/Pumba**: Open-source alternatives

## Automation & Scripting

### 19. Design an automated backup and recovery strategy

    **Backup Types:**
    - **Full Backups**: Complete system backup (weekly)
    - **Incremental**: Changed since last full backup (daily)
    - **Differential**: Changed since last full backup (daily)
    - **Continuous**: Point-in-time recovery

    **Recovery Strategies:**
    - **RTO/RPO Definition**: Recovery time/point objectives
    - **Multi-region Backups**: Cross-region disaster recovery
    - **Immutable Backups**: Tamper-proof backup storage

    **Automation:**
    - **Scheduled Jobs**: Cron jobs, AWS Lambda, Cloud Scheduler
    - **Testing**: Regular restore testing and validation
    - **Monitoring**: Backup success/failure alerts

### 20. How do you ensure code quality in infrastructure code?

    **Testing Strategies:**
    - **Unit Testing**: Individual module/component testing
    - **Integration Testing**: Infrastructure component interaction
    - **End-to-end Testing**: Complete workflow validation

    **Code Quality:**
    - **Linting**: tflint, checkov for Terraform
    - **Security Scanning**: tfsec, checkov vulnerability detection
    - **Policy Checking**: OPA Rego policies, Sentinel
    - **Documentation**: Auto-generated from code comments

    **CI/CD Integration:**
    - Pre-commit hooks for local validation
    - Automated testing on PR creation
    - Gating deployments on quality checks

## Leadership & Organizational Questions

### 21. How do you advocate for DevOps culture in your organization?

    **Organizational Change:**
    - **Education**: Workshops, brown bag sessions
    - **Cross-functional Teams**: Developers + Operations collaboration
    - **Shared Ownership**: SRE/DevOps as team responsibility

    **Technical Enablement:**
    - **Self-service Platforms**: Internal developer portals
    - **Golden Paths**: Curated technology choices and patterns
    - **Documentation**: Runbooks, playbooks, and guides

    **Cultural Transformation:**
    - **Blame-free Culture**: Psychological safety for failure
    - **Automation Mindset**: Manual processes are technical debt
    - **Continuous Learning**: Budget for training and conferences

### 22. Explain site reliability engineering (SRE) principles

    **Core Principles:**
    - **Service Level Objectives**: Reliability targets, not 100%
    - **Error Budgets**: Explicit allowance for failures
    - **Toil Reduction**: Automate repetitive operational work
    - **Blameless Culture**: Focus on system improvement

    **SRE Practices:**
    - **On-call Rotation**: Fair distribution of incident response
    - **Post-mortem Culture**: Learning from every incident
    - **Capacity Planning**: Proactive scaling planning
    - **Gradual Change**: Progressive rollouts, canary deployments

    **Tools and Metrics:**
    - **SLO Monitoring**: Track service reliability against targets
    - **Synthetic Monitoring**: Proactive problem detection
    - **Capacity Planning**: Load testing and forecasting
