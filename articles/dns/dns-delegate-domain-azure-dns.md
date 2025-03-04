---
title: 'Tutorial: Host your domain and subdomain - Azure DNS'
description: In this tutorial, you learn how to configure Azure DNS to host your DNS zones.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: tutorial
ms.date: 05/25/2022
ms.author: rohink
#Customer intent: As an experienced network administrator, I want to configure Azure DNS, so I can host DNS zones.
---

# Tutorial: Host your domain in Azure DNS

You can use Azure DNS to host your DNS domain and manage your DNS records. By hosting your domains in Azure, you can manage your DNS records by using the same credentials, APIs, tools, and billing as your other Azure services.

Suppose you buy the domain `contoso.net` from a domain name registrar and then create a zone with the name `contoso.net` in Azure DNS. Since you're the owner of the domain, your registrar offers you the option to configure the name server (NS) records for your domain. The registrar stores the NS records in the .NET parent zone. Internet users around the world are then directed to your domain in your Azure DNS zone when they try to resolve DNS records in contoso.net.


In this tutorial, you learn how to:

> [!div class="checklist"]
> * Create a DNS zone.
> * Retrieve a list of name servers.
> * Delegate the domain.
> * Verify the delegation is working.


If you don’t have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

## Prerequisites

You must have a domain name available to test with that you can host in Azure DNS. You must have full control of this domain. Full control includes the ability to set the name server (NS) records for the domain.

In this example, we'll reference the parent domain a `contoso.net`.

## Create a DNS zone

1. Go to the [Azure portal](https://portal.azure.com/) to create a DNS zone. Search for and select **DNS zones**.

1. Select **+ Create**.

1. On the **Create DNS zone** page, enter the following values, and then select **Review + create**.

    | **Setting** | **Value** | **Details** |
    |--|--|--|
    | **Resource group**    | *ContosoRG* | Create a resource group. The resource group name must be unique within the subscription that you selected. The location of the resource group doesn't affect the DNS zone. The DNS zone location is always "global," and isn't shown. |
    | **This zone is a child of an existing zone already hosted in Azure DNS**        | leave unchecked | Leave this box unchecked since the DNS zone is **not** a [child zone](./tutorial-public-dns-zones-child.md). |
    | **Name**              | *contoso.net* | Enter your parent DNS zone name      |
    | **Resource group location**          | *East US* | This field is based on the location selected as part of Resource group creation  |
    
1. Select **Create**.


   > [!NOTE] 
   > If the new zone that you are creating is a child zone (e.g. Parent zone = `contoso.net` Child zone = `child.contoso.net`), please refer to our [Creating a new Child DNS zone tutorial](./tutorial-public-dns-zones-child.md)

## Retrieve name servers

Before you can delegate your DNS zone to Azure DNS, you need to know the name servers for your zone. Azure DNS gives name servers from a pool each time a zone is created.

1. Select **Resource groups** in the left-hand menu, select the **ContosoRG** resource group, and then from the **Resources** list, select **contoso.net** DNS zone. 

1. Retrieve the name servers from the DNS zone page. In this example, the zone `contoso.net` has been assigned name servers `ns1-01.azure-dns.com`, `ns2-01.azure-dns.net`, `ns3-01.azure-dns.org`, and `ns4-01.azure-dns.info`:

    :::image type="content" source="./media/dns-delegate-domain-azure-dns/dns-name-servers.png" alt-text="Screenshot of D N S zone showing name servers" lightbox="./media/dns-delegate-domain-azure-dns/dns-name-servers.png":::

Azure DNS automatically creates authoritative NS records in your zone for the assigned name servers.

## Delegate the domain

Once the DNS zone gets created and you have the name servers, you'll need to update the parent domain with the Azure DNS name servers. Each registrar has its own DNS management tools to change the name server records for a domain. 

1. In the registrar's DNS management page, edit the NS records and replace the NS records with the Azure DNS name servers.

1. When you delegate a domain to Azure DNS, you must use the name servers that Azure DNS provides. Use all four name servers, regardless of the name of your domain. Domain delegation doesn't require a name server to use the same top-level domain as your domain.

> [!NOTE]
> When you copy each name server address, make sure you copy the trailing period at the end of the address. The trailing period indicates the end of a fully qualified domain name. Some registrars append the period if the NS name doesn't have it at the end. To be compliant with the DNS RFC, include the trailing period.

Delegations that use name servers in your own zone, sometimes called *vanity name servers*, aren't currently supported in Azure DNS.

## Verify the delegation

After you complete the delegation, you can verify that it's working by using a tool such as *nslookup* to query the Start of Authority (SOA) record for your zone. The SOA record is automatically created when the zone is created. You may need to wait at least 10 minutes after you complete the delegation, before you can successfully verify that it's working. It can take a while for changes to propagate through the DNS system.

You don't have to specify the Azure DNS name servers. If the delegation is set up correctly, the normal DNS resolution process finds the name servers automatically.

1. From a command prompt, enter a nslookup command similar to the following example:

   ```
   nslookup -type=SOA contoso.net
   ```

1. Verify that your response looks similar to the following nslookup output:

   ```
   Server: ns1-04.azure-dns.com
   Address: 208.76.47.4

   contoso.net
   primary name server = ns1-04.azure-dns.com
   responsible mail addr = msnhst.microsoft.com
   serial = 1
   refresh = 900 (15 mins)
   retry = 300 (5 mins)
   expire = 604800 (7 days)
   default TTL = 300 (5 mins)
   ```

## Clean up resources

When no longer needed, you can delete all resources created in this tutorial by following these steps to delete the resource group **ContosoRG**:

1. From the left-hand menu, select **Resource groups**.

2. Select the **ContosoRG** resource group.

3. Select **Delete resource group**.

4. Enter **ContosoRG** and select **Delete**.

## Next steps

In this tutorial, you created a DNS zone for your domain and delegated it to Azure DNS. To learn about Azure DNS and web apps, continue with the tutorial for web apps.

> [!div class="nextstepaction"]
> [Create DNS records for a web app in a custom domain](./dns-web-sites-custom-domain.md)
