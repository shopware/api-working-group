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

*/api/app-system/{appName}/privileges/accept* 

Accept a set of permissions for the specified app name. Requires admin scope and `acl_role:update` permission to write.

*/api/app-system/privileges/accepted*

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
                    "requestedPrivileges": {
                      "type": "object",
                      "additionalProperties": {
                        "type": "object",
                        "additionalProperties": {
                          "type": "array",
                          "items": {
                            "type": "object",
                            "properties": {
                              "extensions": {
                                "type": "array",
                                "items": {}
                              },
                              "entity": {
                                "type": "string"
                              },
                              "operation": {
                                "type": "string"
                              }
                            }
                          }
                        }
                      }
                    }
                  }
                },
                "example": {
                  "requestedPrivileges": {
                    "SwagAnalytics": {
                      "media": [
                        {
                          "extensions": [],
                          "entity": "media",
                          "operation": "read"
                        }
                      ],
                      "settings": [
                        {
                          "extensions": [],
                          "entity": "state_machine",
                          "operation": "read"
                        },
                        {
                          "extensions": [],
                          "entity": "state_machine_state",
                          "operation": "read"
                        }
                      ]
                    }
                  }
                }
              }
            }
          },
          "401": {
            "description": "Unauthorized Access."
          },
          "403": {
            "description": "Forbidden. Insufficient permissions."
          }
        }
      }
    },
    "/app-system/{appName}/privileges/accepted": {
      "get": {
        "tags": [
          "App System"
        ],
        "summary": "Get the accepted privileges for the current app integration",
        "description": "Returns the list of accepted privileges for the current app integration.",
        "operationId": "getAcceptedPrivileges",
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
                    "acceptedPrivileges": {
                      "type": "object",
                      "additionalProperties": {
                        "type": "array",
                        "items": {
                          "type": "object",
                          "properties": {
                            "extensions": {
                              "type": "array",
                              "items": {}
                            },
                            "entity": {
                              "type": "string"
                            },
                            "operation": {
                              "type": "string"
                            }
                          }
                        }
                      }
                    }
                  }
                },
                "example": {
                  "acceptedPrivileges": {
                    "customer": [
                      {
                        "extensions": [],
                        "entity": "customer",
                        "operation": "read"
                      },
                      {
                        "extensions": [],
                        "entity": "customer_group",
                        "operation": "read"
                      }
                    ],
                    "order": [
                      {
                        "extensions": [],
                        "entity": "order",
                        "operation": "read"
                      }
                    ]
                  }
                }
              }
            }
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
    "/app-system/{appName}/privileges": {
      "patch": {
        "tags": [
          "App System"
        ],
        "summary": "Manage privileges for an app",
        "description": "Accepts or revokes specified privileges for the given app. Requires admin scope and `acl_role:update` permission.",
        "operationId": "managePrivileges",
        "security": [
          {
            "oAuth": [
              "admin"
            ]
          }
        ],
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
                "accept": [
                  "customer:read",
                  "order:read"
                ],
                "revoke": [
                  "product:write"
                ]
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
            "description": "Forbidden. Insufficient permissions."
          },
          "404": {
            "description": "App not found."
          }
        }
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
