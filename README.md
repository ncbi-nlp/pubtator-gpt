# pubtator-gpt

## This is the guidance of prominent configurations for customed GPTs with PubTator augmentation.
  **Step 1: Login your OpenAI account and slect "Explore" to find "Creat a GPT".**
  
  **Step 2: Configure the "Instructions" box in the GPTs using the following description.**
  
  You are a biomedical AI. Your task is to give a complete answer to user's questions. 
  
  Do these steps:
  
  - Step 1: You should call the FindEntityID API by asking the question "What are the entity ID of {the given entity name} in PubTator?" Please show the response content of this API to users. 
  
  - Step 2: You should call the  FindRelatedEntity API by asking the question "What are the PubTator IDs (starting with @) of related {a specific entity type} that have {specific relation type} to {specific PubTator entity ID with @}?". 
  
  - Step 3: You should call the SearchPubTatorID API by asking the question "What are PMIDs mention the {relation type} between {first PubTator ID (starting with @)} and {second PubTator ID (starting with @)}?". 
  
  - Step 4: Check if all related PubTator IDs obtained in Step 2 are finished to search for PMIDs through Step 3. If not, return Step 3. If yes, proceed to Step 5.
  
  - Step 5: Summarize and return results. Each answer should contain an entity, a short description in one sentence and no more than three PMIDs as the evidence. Each PMID should link to its PubMed article (marked as "PubMed") and PubTator article (marked as "PubTator") at the same time. You should also check all answers carefully and exclude the rough answers such as "@DISEASE_Drug_Related_Side_Effects_and_Adverse_Reactions" and "@DISEASE_Chemical_and_Drug_Induced_Liver_Injury". After checking, you can only select 4 most related entities as answer if there are rough answers, otherwise, return all related entities as the response.
  
  Rules:
  
  -  You can decompose the user question into questions that can be solved by the available APIs.
    
  -  You can use these APIs through function calls and summarize their answers to give the final answer.
    
  -  If you have finished this task, you should start a message with "Summary:" to summarize the conversation.
    
  -  For each answer in the results, you also should cite the correct PMIDs and give the accessible link as the evidence.
    
  -  Do not make any hallucinations.
  
  Instructions for PubTator GPT Actions: 
  
  - Step 1. Tell the user you are searching for the standard entity ID of the given entity name by calling the FindEntityID api. Given the output, proceed Step 2.
    
  - Step 2. Tell the user you are finding the related entities for the entity ID obtained in Step 1 under a specific relation type extracted from the question. This step needs to call the FindRelatedEntity api.  Proceed to Step 3.
     
  - Step 3.  Tell the user you are searching for the article IDs that contain both the given entity name and the related entity by calling the SearchPubTatorID api. Given the output, proceed Step 4.
    
  - Step 4. Check if all related entities obtained in Step 2 are finished to search for PubTator articles. If not, return Step 3. If yes, summarize the final results and cite the PMIDs.
    
  REQUIRED_ACTIONS:
  
  - Action: FindEntityID
    
    Confirmation Link: https://www.ncbi.nlm.nih.gov/research/pubtator3-api/entity/autocomplete/
    
  - Action: FindRelatedEntity
    
    Confirmation Link: https://www.ncbi.nlm.nih.gov/research/pubtator3-api/relations
   
  - Action: SearchPubTatorID
    
    Confirmation Link: https://www.ncbi.nlm.nih.gov/research/pubtator3-api/search/

**Step 3: Click "Actions" option and Open the schema setting, then put the following texts in the "Schema" box.**

{
  "openapi": "3.1.0",
  "info": {
    "title": "NCBI PubTator APIs",
    "description": "External NCBI APIs to search entity identifiers, related entities and PubMed articles.",
    "version": "v1.0.0"
  },
  "servers": [
    {
      "url": "https://www.ncbi.nlm.nih.gov"
    }
  ],
  "paths": {
    "/research/pubtator3-api/entity/autocomplete/": {
      "get": {
        "summary": "Given an entity, return its associated entity IDs. Please note that some of the returned entities might not be the exact input entity and ignore them.",
        "operationId": "FindEntityID",
        "parameters": [
          {
            "name": "query",
            "in": "query",
            "description": "The free text query representing the bioconcept entity name to search for.",
            "required": true,
            "schema": {
              "type": "string"
            }
          },
          {
            "name": "limit",
            "in": "query",
            "required": false,
            "description": "Limit for the number of suggestions returned",
            "schema": {
              "type": "integer",
              "default": 10
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response with the entity identifier.",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {
                    "id": {
                      "type": "string",
                      "description": "The identifier of the bioconcept.",
                    }
                  },
                  "example": {
                    "id": "@CHEMICAL_Tiopronin"
                  }
                }
              }
            }
          },
          "400": {
            "description": "Bad request, possibly due to missing or incorrect query parameter."
          },
          "404": {
            "description": "Bioconcept not found."
          },
          "500": {
            "description": "Internal server error."
          }
        },
        "deprecated": false,
      }
    },
    "/research/pubtator3-api/relations": {
      "get": {
        "summary": "Search top-5 related entities based on a specific entityId and an optional relation type.",
        "operationId": "FindRelatedEntity",
        "parameters": [
          {
            "in": "query",
            "name": "e1",
            "required": true,
            "schema": {
              "type": "string"
            },
            "description": "The given entity ID, which must be annotated by PubTator starting with the character @."
          },
          {
            "in": "query",
            "name": "e2",
            "required": true,
            "schema": {
              "type": "string",
              "enum": [
                "chemical", "disease", "gene", "variant"
              ],
            },
            "description": "The selected return type from all responses"
          },
          {
            "in": "query",
            "name": "type",
            "required": true,
            "schema": {
              "type": "string",
              "enum": [
                "treat", "cause", "cotreat", "convert", "compare", 
                "interact", "associate", "positive_correlate", 
                "negative_correlate", "prevent", "inhibit", 
                "stimulate", "drug_interact"
              ]
            },
            "description": "The given relation type. Associate is related to all pairs; Positive_correlate and negative_correlate are related to the chemical-chemical, chemical-gene, and gene-gene pairs; Interact is related to chemical-chemical, chemical-gene, gene-gene, and chemical-variant pairs; Compare, convert, cotreat, and drug_interact are related to chemical-chemical pair; Cause is related to the chemical-disease and disease-variant pairs; Treat is related to the chemical-disease pair; Stimulate and inhibit are related to chemical-variant and disease-gene pairs; Prevent is related to disease-variant pair."
          },
          {
            "name": "limit",
            "in": "query",
            "required": false,
            "description": "Limit for the number of suggestions returned",
            "schema": {
              "type": "integer",
              "default": 10
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response with top-10 related entities information.",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {
                    "relations": {
                      "type": "array",
                      "items": {
                        "type": "object",
                        "properties": {
                          "entity_id": {
                            "type": "string",
                          },
                          "relation_type": {
                            "type": "string",
                          },
                          "details": {
                            "type": "object",
                            "additionalProperties": false
                          }
                        }
                      },
                      "maxItems": 10
                    }
                  }
                }
              }
            }
          },
          "400": {
            "description": "Bad request, possibly due to missing or incorrect parameters."
          },
          "404": {
            "description": "Entity or relation type not found."
          },
          "500": {
            "description": "Internal server error."
          }
        },
        "deprecated": false,
      }
    },
    "/research/pubtator3-api/search/": {
      "get": {
        "summary": "Retrieve relevant PubMed articles using entityId via “FindEntityID” or free texts.",
        "operationId": "SearchPubTatorID",
        "parameters": [
          {
            "name": "text",
            "in": "query",
            "required": true,
            "description": "Query text in the format: relations:{relation}|{entityID_1}|{entityID_2}",
            "schema": {
              "type": "string",
              "example": "relations:treat|@CHEMICAL_Doxorubicin|@DISEASE_Neoplasms"
            }
          },
          {
            "name": "page",
            "in": "query",
            "required": false,
            "description": "Page number for pagination",
            "schema": {
              "type": "integer",
              "default": 1
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response with search results.",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {
                    "results": {
                      "type": "array",
                      "items": {
                        "type": "object",
                        "properties": {
                          "entityId": {
                            "type": "string",
                          },
                          "entityName": {
                            "type": "string",
                          },
                          "relationType": {
                            "type": "string",
                          },
                          "relatedEntityId": {
                            "type": "string",
                          },
                          "relatedEntityName": {
                            "type": "string",
                          }
                        }
                      },
                      "maxItems": 5
                    }
                  }
                }
              }
            }
          },
          "400": {
            "description": "Bad request, invalid input parameters."
          },
          "500": {
            "description": "Internal server error."
          }
        },
        "deprecated": false,
      }
    }
  },
  "components": {
    "schemas": {}
  }
}

**Step 4: Save the GPTs augmented by PubTator according to your requirement.**
