```javascript
/*
  Case: GeneDX enters a case with a candidate gene, NOTCH2, and wants
  to see other cases where that gene is also listed as a candidate. If the
  case is specifically listed as a species other than human, it should be
  ignored.
*/

{
  "meta": {
    "dimensions": [
      "mandatory": {
        "version": 1
      },
      "patient": {
        "version": 1
      },
      "species": {
        "version": 1
      },
      "genes": {
        "version": 1
      }
    ]
  },

  "patient": {
    "meta": {
      "id": "457",
      "contact": {
        "name": "GeneDX",
        "institution": "GeneDX",
        "href": "mailto:match@genedx.com"
      }
    },

    "species": {
      "id": "NCBITaxon:9606",
      "label": "Homo sapiens"
    },

    "genes": {
      "items": [
        {
          "ids": [
            {
              "id": "Ensembl:ENSG00000134250",
              "label": "NOTCH2"
            }
          ]
        }
      ]
    }
  },

  "query": {
    "dimensions": {
      "mandatory": {
        // Require genes to be supported and present
        "items": [
          "genes"
        ]
      }

      // If genes is supported, require NOTCH2
      "genes": {
        "items": [
          {
            "ids": [
              {
                "id": "Ensembl:ENSG00000134250"
              }
            ]
          }
        ]
      },

      // If sex is supported, require human
      "sex": {
        "id": "NCBITaxon:9606"
      }
    }
  }
}
```
