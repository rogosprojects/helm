# Helm Charts Release Notes

This file contains release notes for the Helm charts in this repository. Each chart's releases are documented separately.

## kpods-monitor

### Latest Changes

- Removed `refreshInterval` parameter as the application now uses Kubernetes Informers for improved efficiency and real-time updates
- Configuration has been simplified with the removal of polling-related settings

### Previous Releases

- Initial chart release with support for monitoring Kubernetes applications and pods

## klogs-viewer 

### Latest Changes

- Initial chart release with support for viewing and searching Kubernetes logs

## General Updates

- Charts follow Helm best practices
- Documentation improvements across all charts
- Enhanced RBAC configurations