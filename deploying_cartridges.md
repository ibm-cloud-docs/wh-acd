---

copyright:
  years: 2011, 2019
lastupdated: "2019-04-12"

keywords: annotator clinical data, clinical data, annotation

subcollection: wh-acd

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:tip: .tip}
{:important: .important}
{:note: .note}
{:deprecated: .deprecated}
{:pre: .pre}
{:codeblock: .codeblock}
{:screen: .screen}
{:javascript: .ph data-hd-programlang='javascript'}
{:java: .ph data-hd-programlang='java'}
{:python: .ph data-hd-programlang='python'}
{:swift: .ph data-hd-programlang='swift'}

# Cartridge Deployment 
{: #deploy_cartridges}

1. The consumer uses the {{site.data.keyword.wh-acd_short}} Configuration Editor to create a new cartridge (or modify an existing one) and customizes the contents (artifacts) of the cartridge to their domain. After that, the consumer will **Export** the cartridge in order to save a snapshot of the cartridge.
2. The consumer deploys the cartridge snapshot (a zip file) to  {{site.data.keyword.wh-acd_short}} using _POST /v1/cartridges_ API. A successful request for creating a cartridge will return with HTTP <q>202 ACCEPTED</q> response code and will include the path to the resource, e.g., /v1/cartridges/cartridge_id in the response body and the response header. The resource path can be used in _GET /v1/cartridges/catridgeId_ API to obtain the overall deployment status. In the following curl example, the consumer's cartridge file is `/path/to/name_of_cartridge_file.zip`.

    ```Curl
 curl -X POST -u "apikey":"{apikey}" \
  --header "Accept:application/json" \
 --header "Content-Type:application/octet-stream" \
 --data-binary @/path/to/name_of_cartridge_file.zip \
 "{url}/v1/cartridges?vesion=2019-09-01"
    ```
    Multiple deployments of the same cartridge should use the _POST /v1/cartridges_ operation for the creation and, subsequently, use the _PUT /v1/cartridges_ operation for the update. Deployment of the same cartridge using the POST operation multiple times will result in HTTP <q>409 CONFLICT</q> response code .
    
3. The consumer updates the cartridge snapshot using the _PUT /v1/cartridges_ API. The cartridges id is not required in the path parameter and is extracted directly from the zip file. A successful request for updating the cartridge deployment will result in HTTP <q>202 ACCEPTED</q> response code and will include the path to the resource, e.g., /v1/cartridges/cartridge_id in the response body and the response header.  

    ```Curl
curl -X PUT -u "apikey":"{apikey}" \
--header "Accept:application/json" \
--header "Content-Type:application/octet-stream" \
--data-binary @/path/to/name_of_cartridge_file.zip \
"{url}/v1/cartridges?vesion=2019-09-01"
    ```
    
    Updating a cartridge deployment assumes that the cartridge has been previously deployed. The consumer creates an initial deployment with the POST operation and subsequent deployment uses the PUT operation. Updating a cartridge deployment with missing or empty deployment status will return with HTTP <q>404 Not Found</q> response code.

4.  The consumer retrieves all deployment status using _GET /v1/cartridges_ API, which include all cartridge deployed for a specific tenant. An empty list is returned if no catridge has been deployed.  

    ```Curl
curl -X GET -u "apikey":"{apikey}" \
--header "Accept:application/json" \
"{url}/v1/cartridges?version=2019-09-01"
    ```

5.  The consumer can view the contents of a specific deployment by invoking the _GET /v1/cartridges_ API with the cartridge ID supplied as the path parameter . If the supplied ID does not exists, then a HTTP  <q> 404 Not Found </q> response code is returned. The following curl command returns the deployment status of the <q>cartridge_id</qt>.

    ```Curl
curl -X GET -u "apikey":"{apikey}" \
--header "Accept:application/json" \
"{url}/v1/cartridges/cartridge_id?version=2019-09-01"
    ```

Replace `{apikey}` and `{url}` with the actual API key and URL in all sample codes above.

> The _/v1/catridges_ API is the recommended way for a cartridge deployment and is compatible with the legacy _/v1/deploy_ API. In many situations, the consumer has already deployed a cartridge using the _/v1/deploy_ API and the consumer can immediately update the same cartridge using the above POST and PUT operations on the _/v1/catridges_ API to initially create and to subsequently update the cartridge deployment. 
> 
> A typical _POST /v1/catridges_ operation creates and initializes a deployment for cartridge that has never been deployed to the system. For cartridges that have been previously deployed with the _/v1/deploy_ API, the _POST /v1/cartridges_ API will create, initialize, and update an existing cartridge deployment. Subsequent update to the cartridge can use the _PUT /v1/cartridges_ API. 



# Legacy Cartridge Deployment


1.  The consumer uses the {{site.data.keyword.wh-acd_short}} Configuration Editor to create a new cartridge (or modify an existing one) and customizes the contents (artifacts) of the cartridge to their domain. After that, the consumer will **Export** the cartridge in order to save a snapshot of the cartridge.
2.  The consumer deploys the cartridge snapshot (a zip file) to  {{site.data.keyword.wh-acd_short}} using _POST /v1/deploy_ API. In the following curl example, the consumer's cartridge file is `./my_cartridges/name_of_cartridge_file.zip`, and `update=false` means do not update the resource if it already exists. Specifying the **update=true** parameter on the deploy API to update an existing cartridge.

    Replace `{apikey}` and `{url}` with the actual API key and URL in the following example. A successful cartridge deploy will result in <q>201 Created</q> if it is a new resource, or <q>200 OK</q> if `update=true` was specified and the existing resource was updated.

    ```Curl
  curl -X POST -u "apikey:{apikey}" \
  --header “Content-Type: application/octet-stream” \
  --header "Accept: application/json" \
  --data-binary @./my_cartridges/name_of_cartridge_file.zip \
  "{url}/v1/deploy?update=false&version=2018-01-17"
```

> Some large cartridge deployments can exceed the request timeout thresholds defined in the DataPower gateways (usually after 2 mins). In that event, you may receive the following error response. See [Cartridge Deployment Timeout](wh-acd?topic=wh-acd-troubleshoot#troubleshoot_deploy_timeout) for additional considerations during the deployment of large cartridges. 