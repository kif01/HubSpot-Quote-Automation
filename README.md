# HubSpot-Quote-Automation
This is a proof of concept that shows to use HubSpot Quote's API to automatically generate quotes and send them to customers based on a specific criteria that gets triggered in a workflow.

# High-Level Flow

<img width="1344" alt="Screenshot 2022-05-04 at 12 14 23" src="https://user-images.githubusercontent.com/15332386/166671546-63bd252b-2688-423f-b2ac-3f2a8ba8c68f.png">

# Workflow Code

```python
import os
import requests
from hubspot import HubSpot
from hubspot.crm.contacts import ApiException

def main(event):

  hubspot = HubSpot(api_key=os.getenv('HAPIKEY'))
  headers = {'accept': 'application/json'}
  base_url = "https://api.hubapi.com/crm/v3/objects/"
  HAPI = os.getenv("HAPIKEY")
  deal_id = event.get('inputFields').get('hs_deal_id')
  quote_template_id = "97424611854" #id of the template that I will be using in this example
  
  #Create Quote
  querystring = {"hapikey":HAPI}
  quote_body= {"properties":{"hs_title": "API Quote 4","hs_expiration_date": "2023-02-22"}}
  
  response = requests.request("POST",base_url+"QUOTE", json=quote_body,headers=headers, params=querystring).json()
  quote_id = response["id"]
  
  #Associate Quote to Deal
  string_url1 = "DEAL/"+deal_id+"/associations/"+"QUOTE/"+quote_id+"/DEAL_TO_QUOTE"
  requests.request("PUT",base_url+string_url1,headers=headers, params=querystring)
  
  #Associate Quote to Template
  string_url2 = "QUOTE_TEMPLATE/"+quote_template_id+"/associations/"+"QUOTE/"+quote_id+"/QUOTE_TEMPLATE_TO_QUOTE"
  requests.request("PUT",base_url+string_url2,headers=headers, params=querystring)
  
  #Create Line Item from Product
  product_id = "1363716935" # For the example I'm using a static product to create my line item
  line_item_body= {"properties":{"hs_product_id": product_id, "quantity":1}}
  response = requests.request("POST",base_url+"LINE_ITEM", json=line_item_body,headers=headers, params=querystring).json()
  line_item_id = response["id"]
  
  #Associate Quote to Line Item
  string_url3 = "LINE_ITEM/"+line_item_id+"/associations/"+"QUOTE/"+quote_id+"/LINE_ITEM_TO_QUOTE"
  requests.request("PUT",base_url+string_url3,headers=headers, params=querystring)
  
  #Associate Quote to Contact
  contact_id = "101" #For the example I'm using a static contact
  string_url4 = "CONTACT/"+contact_id+"/associations/"+"QUOTE/"+quote_id+"/CONTACT_TO_QUOTE"
  requests.request("PUT",base_url+string_url4,headers=headers, params=querystring)
  
  #Associate Quote to Company
  company_id = "7397257025" #For the example I'm using a static company
  string_url5 = "COMPANY/"+company_id+"/associations/"+"QUOTE/"+quote_id+"/COMPANY_TO_QUOTE"
  requests.request("PUT",base_url+string_url5,headers=headers, params=querystring)
  
  #Update Quote info
  quote_update_body = {"properties":{"hs_currency": "USD", "hs_sender_firstname": "Khalil", "hs_sender_lastname": "Faraj", "hs_sender_email": "kfaraj@hubspot.com"}}
  requests.request("PATCH",base_url+"QUOTE/"+quote_id, json=quote_update_body,headers=headers, params=querystring)
  
  #Publish Quote
  state_body = {"properties":{"hs_status":"APPROVAL_NOT_NEEDED"}}
  requests.request("PATCH",base_url+"QUOTE/"+quote_id, json=state_body,headers=headers, params=querystring)
  
  #Get Quote Generated Link
  props= "hs_domain,hs_slug"
  querystring = {"hapikey":HAPI, "properties":props}
  response=requests.request("GET",base_url+"QUOTE/"+quote_id,headers=headers, params=querystring).json()
  domain = response["properties"]["hs_domain"]
  slug = response["properties"]["hs_slug"]
  quote_link = domain+"/"+slug

  return {
    "outputFields": {
      "Link": quote_link
     
    }
  }
```
