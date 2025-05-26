# API Change Proposal

## Overview

**Title:** Add Admin API to fetch app permissions and approve them.

## Description

We are in the process of decoupling app updates from the permission review process. Therefore, we need an API to fetch the permissions in review and approve them. We also need a way for an app to fetch it's currently approved set of permissions, so it knows which actions it is allowed to perform.

## Motivation

The change is neccessary to allow the shop owner to approve permission requests. For example, they will recieve a notification that an app has been updated and it requires new permissions. They will be guided to extension page where they can review and accept the new permissions.

On the other hand the app backend needs to know which permissions it has so it can check before performing actions, so we will need a new API to fetch the set of approved permissions.

## Proposed Changes

### API Endpoints

*/api/app-system/privileges/requested*

Fetch the set of requested permissions. It retrieves all requested permissions for all apps. Requires admin scope and `acl_role:read` permission to read. 

*/api/app-system/{appName}/privileges*

Update the set of accepted permissions for a specific app. This endpoint allows the admin to approve or revoke requested permissions for a specific app. It requires admin scope and `acl_role:update` permission to write.

*/api/app-system/{appName}/privileges/accepted*

Fetch the set of accepted permissions for the current integration. Requires admin scope with an integration.

### Schema

```json
{
  "openapi": "3.0.0",
  "info": {
    "title": "App System API",
    "version": "v1"
  },
  "paths": {
    "/app-system/privileges/requested": {
      "get": {
        "tags": [
          "App System"
        ],
        "summary": "Get requested privileges for all apps",
        "description": "Returns the list of requested privileges for all apps. Requires admin scope and `acl_role:read` permission to read.",
        "operationId": "getRequestedPrivileges",
        "security": [
          {
            "oAuth": [
              "admin"
            ]
          }
        ],
        "responses": {
          "200": {
            "description": "A JSON object containing requested privileges.",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {
                    "privileges": {
                      "type": "object",
                      "additionalProperties": {
                        "type": "array",
                        "items": {
                          "type": "string"
                        }
                      }
                    }
                  }
                },
                "example": {
                  "privileges": {
                    "SwagAnalytics": ["customer:read", "order:read"],
                    "SwagExample": ["product:write"]
                  }
                }
              }
            }
          },
          "400": {
            "description": "Malformed request."
          },
          "401": {
            "description": "Unauthorized Access."
          },
          "403": {
            "description": "Forbidden. Not a valid integration source."
          }
        }
      }
    },
    "/app-system/{appName}/privileges/accepted": {
      "get": {
        "tags": [
          "App System"
        ],
        "summary": "Get accepted privileges for an app",
        "description": "Returns the list of accepted privileges for the current integration. Requires admin scope with an integration.",
        "operationId": "getAcceptedPrivileges",
        "security": [
          {
            "oAuth": [
              "admin"
            ]
          }
        ],
        "responses": {
          "200": {
            "description": "A JSON object containing accepted privileges.",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {
                    "privileges": {
                      "type": "object",
                      "additionalProperties": {
                        "type": "boolean"
                      }
                    }
                  }
                },
                "example": {
                  "privileges": {
                    "customer.read": true,
                    "order.read": true
                  }
                }
              }
            }
          },
          "400": {
            "description": "Malformed request."
          },
          "401": {
            "description": "Unauthorized Access."
          },
          "403": {
            "description": "Forbidden. Not a valid integration source."
          },
          "404": {
            "description": "App not found."
          }
        }
      }
    },
    "/app-system/{appName}/privileges": {
      "patch": {
        "tags": ["App System"],
        "summary": "Accept or revoke privileges for an app",
        "description": "Accepts or revokes specified privileges for the given app.",
        "operationId": "managePrivileges",
        "parameters": [
          {
            "name": "appName",
            "in": "path",
            "required": true,
            "schema": {
              "type": "string"
            }
          }
        ],
        "requestBody": {
          "required": true,
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "properties": {
                  "accept": {
                    "type": "array",
                    "items": {
                      "type": "string"
                    }
                  },
                  "revoke": {
                    "type": "array",
                    "items": {
                      "type": "string"
                    }
                  }
                }
              },
              "example": {
                "accept": ["customer:read", "order:read"],
                "revoke": ["product:write"]
              }
            }
          }
        },
        "responses": {
          "204": {
            "description": "Returns no content if privileges were managed successfully."
          },
          "400": {
            "description": "Malformed request."
          },
          "401": {
            "description": "Unauthorized Access."
          },
          "403": {
            "description": "Forbidden. Not a valid integration source."
          },
          "404": {
            "description": "App not found."
          }
        },
        "security": [
          {
            "oAuth": [
              "admin"
            ]
          }
        ]
      }
    }
  },
  "components": {
    "securitySchemes": {
      "oAuth": {
        "type": "oauth2",
        "description": "Authentication using OAuth 2.0",
        "flows": {
          "password": {
            "tokenUrl": "/api/oauth/token",
            "scopes": {
              "admin": "Admin scope for administrative operations"
            }
          }
        }
      }
    }
  }
}
```
