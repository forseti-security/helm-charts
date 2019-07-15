# Forseti Security Helm Charts

This repository contains the Helm charts for the Forseti Security applications.

## Installing Charts from this Repository

Add the Repository to Helm:

    helm repo add forseti-security https://forseti-security-charts.storage.googleapis.com/release

Install Forseti:

    helm install forseti-security/forseti-security

## Apps and Compatibility

| Application                          | Application Version | Compatible Chart Version |
| -----------------                    | ------------------  | ---------------          |
| [Forseti](./charts/forseti-security) | 2.18.0              | 1.0.0 |
