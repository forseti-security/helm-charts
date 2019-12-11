# Forseti Security Helm Charts

This repository contains the Helm charts for the Forseti Security applications.

## Installing Charts from this Repository

Add the Repository to Helm:

    helm repo add forseti-security https://forseti-security-charts.storage.googleapis.com/release

Please see each application's README.md in the charts/ directory for instructions on how to install the chart.

## Apps and Compatibility

| Application                          | Application Version | Compatible Chart Version |
| -----------------                    | ------------------  | ---------------          |
| [Forseti](./charts/forseti-security) | 2.18.0<br />2.19.0<br />2.19.1<br />2.20.0<br />2.21.0<br />2.22.0<br />2.23.0 <br />2.24.0 <br />2.25.0 | 1.0.0, 1.0.1<br />1.1.0<br />1.1.0<br />1.1.0<br />1.2.0<br />1.3.0, 2.0.0<br />2.0.0<br />2.1.0<br />2.2.0 |
| [Config Validator](./charts/config-validator) | latest<br />572e207 | 0.0.1, 0.1.0<br />0.1.0, 0.2.0 |
