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
    "openapi": "3.1.0",
    "info": [],
    "paths": {
        "/app-system/privileges/requested": {
            "get": {
                "tags": ["App System"],
                "summary": "Get requested privileges",
                "description": "Returns the list of requested privileges for all apps.",
                "operationId": "getRequestedPrivileges",
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
                                                {"extensions": [], "entity": "media", "operation": "read"}
                                            ],
                                            "settings": [
                                                {"extensions": [], "entity": "state_machine", "operation": "read"},
                                                {"extensions": [], "entity": "state_machine_state", "operation": "read"}
                                            ]
                                        }
                                    }
                                }
                            }
                        }
                    },
                    "500": {
                        "description": "Incorrect source. "
                    }
                }
            }
        },
        "/app-system/privileges/accepted" : {
            "get": {
                "tags": ["App System"],
                "summary": "Get the accepted privileges for the current app integration",
                "description": "Returns the list of accepted privileges for the current app integration.",
                "operationId": "getAcceptedPrivileges",
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
                                            {"extensions": [], "entity": "customer", "operation": "read"},
                                            {"extensions": [], "entity": "customer_group", "operation": "read"}
                                        ],
                                        "order": [
                                            {"extensions": [], "entity": "order", "operation": "read"}
                                        ]
                                    }
                                }
                            }
                        }
                    },
                    "500": {
                        "description": "Incorrect source. "
                    }
                }
            }
        },
        "/app-system/{appName}/privileges/accept": {
            "post": {
                "tags": ["App System"],
                "summary": "Accept privileges for an app",
                "description": "Accepts specified privileges for the given app.",
                "operationId": "acceptPrivileges",
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
                                "type": "array",
                                "items": {
                                    "type": "string"
                                }
                            },
                            "example": ["customer:read", "order:read"]
                        }
                    }
                },
                "responses": {
                    "204": {
                        "description": "Returns no content if privileges were accepted successfully."
                    },
                    "404": {
                        "description": "App not found."
                    },
                    "400": {
                        "description": "Malformed request."
                    },
                    "500": {
                        "description": "Incorrect source. "
                    }
                }
            }
        }
    }
}
```
