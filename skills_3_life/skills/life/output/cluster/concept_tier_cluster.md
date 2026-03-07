# Concept: Tier Cluster — Dynamic Negotiation (N=3)
## Cascade: Flash → Reference → Memory. At each tier: if confidence ≥ threshold, return.
## Thresholds per rootfs category (from clusters.yml): SECURITY=0.90/0.95, default=0.75/0.85, PERFORMANCE=0.60/0.70.
## CPU cache isomorphism: L1=Flash, L2=Reference, L3=Memory. Miss cascade identical.
