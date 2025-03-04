---
title: Diagnose and troubleshoot provisioning errors with Azure IoT Hub DPS
description: Learn to diagnose and troubleshoot common errors for Azure IoT Hub Device Provisioning Service (DPS)
author: kgremban
ms.service: iot-dps
services: iot-dps
ms.topic: conceptual
ms.date: 05/25/2022
ms.author: kgremban
#Customer intent: As an operator for Azure IoT Hub DPS, I need to know how to find out when devices are not being provisioned and troubleshoot and resolve those issues right away.
---

# Troubleshooting with Azure IoT Hub Device Provisioning Service

Provisioning issues for IoT devices can be difficult to troubleshoot because there are many possible points of failures such as attestation failures, registration failures, etc. This article provides guidance on how to detect and troubleshoot device provisioning issues via Azure Monitor. To learn more about using Azure Monitor with DPS, see [Monitor Device Provisioning Service](monitor-iot-dps.md).

## Using Azure Monitor to view metrics and set up alerts

To view and set up alerts on IoT Hub Device Provisioning Service metrics:

1. Sign in to the [Azure portal](https://portal.azure.com).

2. Browse to your IoT Hub Device Provisioning Service.

3. Select **Metrics**.

4. Select the desired metric. For supported metrics, see [Metrics](monitor-iot-dps-reference.md#metrics).

5. Select desired aggregation method to create a visual view of the metric.

6. To set up an alert of a metric, select **New alert rules** from the top right of the metric blade, similarly you can go to **Alert** blade and select **New alert rules**.

7. Select **Add condition**, then select the desired metric and threshold by following prompts.

To learn more about viewing metrics and setting up alerts on your DPS instance, see [Analyzing metrics](monitor-iot-dps.md#analyzing-metrics) and [Alerts](monitor-iot-dps.md#alerts) in Monitor Device Provisioning Service.

## Using Log Analytics to view and resolve errors

1. Sign in to the [Azure portal](https://portal.azure.com).

2. Browse to your Device Provisioning Service.

3. Select **Diagnostics settings**.

4. Select **Add diagnostic setting**.

5. Configure the desired logs to be collected. For supported categories, see [Resource logs](monitor-iot-dps-reference.md#resource-logs).

6. Tick the box **Send to Log Analytics** ([see pricing](https://azure.microsoft.com/pricing/details/log-analytics/)) and save.

7. Go to **Logs** tab in the Azure portal under Device Provisioning Service resource.

8. Write **AzureDiagnostics** as a query and click **Run** to view recent events.

9. If there are results, look for `OperationName`, `ResultType`, `ResultSignature`, and `ResultDescription` (error message) to get more detail on the error.

## Common error codes

Use this table to understand and resolve common errors.

| Error Code| Description | HTTP Status Code |
|-------|------------|------------|
| 400 | The body of the request is not valid; for example, it cannot be parsed, or the object cannot be validated.| 400 Bad format |
| 401 | The authorization token cannot be validated; for example, it is expired or does not apply to the request’s URI. This error code is also returned to devices as part of the TPM attestation flow. | 401 Unauthorized|
| 404 | The Device Provisioning Service instance, or a resource (e.g. an enrollment) does not exist. |404 Not Found |
| 412 | The ETag in the request does not match the ETag of the existing resource, as per RFC7232. | 412 Precondition failed |
| 429 | Operations are being throttled by the service. For specific service limits, see [IoT Hub Device Provisioning Service limits](../azure-resource-manager/management/azure-subscription-service-limits.md#iot-hub-device-provisioning-service-limits). | 429 Too many requests |
| 500 | An internal error occurred. | 500 Internal Server Error|

If an IoT Edge device fails to start with error message `failed to provision with IoT Hub, and no valid device backup was found dps client error.`, see [DPS Client error](/azure/iot-edge/troubleshoot-common-errors?view=iotedge-2018-06&preserve-view=true#dps-client-error) in the IoT Edge (1.1) documentation.

## Next Steps

- To learn more about using Azure Monitor with DPS, see [Monitor Device Provisioning Service](monitor-iot-dps.md).

- To learn about metrics, logs, and schemas emitted for DPS in Azure Monitor, see [Monitoring Device Provisioning Service data reference](monitor-iot-dps-reference.md).
