# Overview #
This is a very rough mockup based on the discussions on the 20 Oct tech call.

Here are the high-level goals:
1. To make the API more extendable through versioned component/dimension handlers, each of which have relatively limited scope over the overall query (e.g. `sex`, `species`, `ageOfOnset`, `genes`, `features`, `variants`, `genome`). Currently, a handler `HANDLER` would read patient data from `.patient.HANDLER`, it would have metadata in `.meta.dimensions.HANDLER`, and it would read query parameters and constraints from `.query.dimensions.HANDLER`.

2. To keep serialization and parsing simple. Only `.meta.dimensions` and `.query.dimensions.mandatory` need to be parsed in full to ensure all mandatory dimensions are handled. After that, those handlers and any others can read only their relevant portions of the overall data structure.

3. To support 3 basic levels of constraint:
  1. Optional: the default for all data in the `.patient` object.
  2. Match or NULL: E.g., "if the case has gene data, TTN must be one of the listed genes." This is specified by providing filter/constraint/weighting information to the particular handler at `.query.dimensions.HANDLER`. If the handler is not supported by the endpoint, this information will be silently ignored.
  3. Match: E.g. "only show me cases with TTN listed". The handler must be present, the data must not be NULL, and data must match. This is implemented by listing the handler in `.query.dimensions.mandatory` and providing the constraints as in the "Match or NULL" case. This has a clear interpretation for handlers that perform discrete filtering, such as `genes`, but `features` may work best in a fuzzy sense and then we need to settle on what mandatory means there.

Design decisions:
* Keep the core of the API a case being matched-by-example
* Avoid mini-languages wherever possible (simply repeating data in `.patient` and `.query` as necessary to avoid needing something like jsonpath cross-references)
* Single `id` fields were almost always replaced by an `ids` list of objects with `id` fields to allow multiple IDs to be specified. An alternative approach would be to specify a primary ID and a list of aliases.
* Components plug into the data model in a standard way and are independently versioned, allowing their formats and semantics to evolve independent of the backbone of the API. For example, the `genes` and `sex` components likely would perform discrete filtering, whereas the `features` component would likely perform some sort of weighted boosting, and the `genome` component would apply a set of variant filters.


# /match request #
```javascript
{
  "meta": {
      // Define the dimensions present in the data and the associated versions
      // Each dimension has a separate specification and they can evolve separately
      // Versions are incremented when backwards-incompatible changes are made to
      //    that dimension's specification
    "dimensions": [
      // "mandatory" would behave like any other component, and validates that the
      // necessary components are supported and present (non-null) in the matching case
      "mandatory": {
        "version"
      },
      "patient": {
        "version"
      },
      "sex": {
        "version"
      },
      "..."
    ]
  },

  "patient": {
    "meta": {
      "id",
      "label",
      "contact": {
        "name",
        "institution",
        "href"
      }
    },

    "sex": {
      "value"
    },

    "species": {
      "id"
    },

    "genes": {
      "items": [
        {
          "ids": [
            {
              "id",
              "label"
            }
          ]
          "zygosity",
          "consequences": [
            {
              "id": "ENSG......",
              "label"
            }
          ]
        }
      ]
    },

    "features": {
      "items": [
        {
          "ids": [
            {
              "id": "HP:.......",
              "label"
            }
          ],
          "ageOfOnset": {
            "id",
            "label",
          },
          "observed"
        }
      ]
    },

    "ageOfOnset": {
      "items": [
        {
          "ids": [
            {
              "id": "HP:....",
              "label"
            }
          ]
        }
      ]
    }

    "disorders": {
      "items": [
        {
          "ids": [
            {
              "id": "MIM:......",
              "label"
            }
          ]
        }
      ]
    },

    "variants": {
      "assembly",
      "items": [
        {
          "referenceName",
          "start",
          "end",
          "referenceBases",
          "alternateBases",
          "zygosity"
        }
      ]
    }
  },

  "query": {
    // Each dimension has a separate JSON object with the handler's name as the key
    "dimensions": {
      "mandatory": {
        "items": [
          "genes"
        ]
      },

      "features": {
        "items": [
          {
            "ids": [
              {
                "id"
              }
            ],
            "weight"
          },
          {
            "ids": [
              {
                "id"
              }
            ],
            "ageOfOnset": {
              "id"
            },
            "weight"
          }
        ]
      },

      "ageOfOnset": {
        "items": [
          {
            "ids": [
              {
                "id"
              }
            ]
          }
        ]
      },

      "sex": {
        "id"
      },

      "genes": {
        "items": [
          {
            "ids": [
              {
                "id"
              }
            ],
            "zygosity"
          }
        ]
      },

      "genome": {
        "filters": [
          {
            "source": "ExAC",
            "annotation": "alleleFrequency"
            "filter": "<",
            "value": 0.01
          }
        ]
      }
    ]
  }
}
```

# /match response #
```javascript
{
  "messages": [
    // any messages that need to be provided to the end user
    {
      "message"
    }
  ],
  "request": {
    "patient": {
      // Parsed patient here
      // Identifiers need to be converted or dropped
      // Variants need to be lifted over or dropped
    },
    "query": {
      // Parsed query here
      // Mandatory items must fully parse,
      // Otherwise, an error should be returned instead
    }
  },
  "results": [
    {
      // Component scores, as necessary
      "scores": {
        "patient",
        "sex",
        "species",
        "ageOfOnset",
        "modeOfInheritance",
        "genes",
        "features",
        "variants",
        "genome"
      },

      "patient": {
        "meta": {
          "..."
        },

        "sex": {
          "..."
        },

        "species": {
          "..."
        }

        "genes": {
          "..."
        },

        "features": {
          "..."
        }
      }
    }
  ]
}
```
