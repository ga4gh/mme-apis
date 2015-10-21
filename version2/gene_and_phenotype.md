```javascript
/*
  Case: Dr. Doe enters a patient with a candidate gene (SRCAP) and a few
  phenotypes (short stature, triangular face, and expressive language delay).
  The patient also presents with diabetes, which the clinician thinks is unrelated.
*/

{
  "meta": {
    "dimensions": [
      "patient": {
        "version": 1
      },
      "genes": {
        "version": 1
      },
      "features": {
        "version": 1
      }
    ]
  },

  "patient": {
    "meta": {
      "id": "P0000123",
      "contact": {
        "name": "Dr. Doe",
        "href": "mailto:jdoe@example.com"
      }
    },

    "species": {
      "id": "NCBITaxon:9606",
      "label": "Homo sapiens"
    },

    "features": {
      "items": [
        {
          "ids": [
            {
              "id": "HP:0004322",
              "label": "Short stature"
            }
          ]
        },
        {
          "ids": [
            {
              "id": "HP:0000325",
              "label": "Triangular face"
            }
          ]
        },
        {
          "ids": [
            {
              "id": "HP:0002474",
              "label": "Expressive language delay"
            }
          ]
        },
        {
          "ids": [
            {
              "id": "HP:0000819",
              "label": "Diabetes mellitus"
            }
          ]
        }
      ]
    },

    "genes": {
      "items": [
        {
          "ids": [
            {
              "id": "Ensembl:ENSG00000080603",
              "label": "SRCAP"
            }
          ]
        }
      ]
    }
  },

  "query": {
    "dimensions": {
      // If phenotype is supported, weight the phenotypes
      "features": {
        "items": [
          // Give twice the normal weight to the three most relevant terms
          {
            "ids": [
              {
                "id": "HP:0004322"
              }
            ],
            "weight": 2
          },
          {
            "ids": [
              {
                "id": "HP:0000325"
              }
            ],
            "weight": 2
          },
          {
            "ids": [
              {
                "id": "HP:0002474"
              }
            ],
            "weight": 2
          },
          {
            "ids": [
              {
                "id": "HP:0000819",
              }
            ],
            // Give half the normal weight to diabetes
            "weight": 0.5
          }
        ]
      }

      // If genes is supported, require SRCAP
      "genes": {
        "items": [
          {
            "ids": [
              {
                "id": "Ensembl:ENSG00000080603"
              }
            ]
          }
        ]
      }
    }
  }
}
```
