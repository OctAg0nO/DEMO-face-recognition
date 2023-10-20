# [Face Recognition](https://github.com/weaviate-tutorials/DEMO-face-recognition#face-recognition)

This project's origin is [here](https://github.com/weaviate/weaviate-examples/tree/main/face-recognition-app)

## [Description](https://github.com/weaviate-tutorials/DEMO-face-recognition#description)

This example application spins up a Weaviate instance using the [img2vec-neural](https://github.com/semi-technologies/i2v-pytorch-models) module, imports a few sample images (you can add your own images, too!) and provides a very simple search frontend in [React](https://reactjs.org/) using the [Weaviate JS Client](https://www.semi.technology/developers/weaviate/current/client-libraries/javascript.html)

(TO DO: Add demo video)

### [Used technology stack](https://github.com/weaviate-tutorials/DEMO-face-recognition#used-technology-stack)

This demo uses the [RESNET50](https://pytorch.org/hub/nvidia_deeplearningexamples_resnet50/) model from [pytorch.org](https://pytorch.org/).

## Need for this Application
Facial Recognition has been an amazing catalyst for advancements in the field of Object Detection and Computer Vision. It has been seamlessly weaved into our everyday life from using FaceID to unlock your phone, to automated album segregation based on individuals in a picture. It is even used in Institutions for marking attendance. 

## Deep Dive into our Solution
Previously, it used to cost a lot of resources to run Facial Recognition Models and perform Image Searches. There were no Vector Databases, remember :)

Now, Vector Databases, like Weaviate do almost all the heavy lifting. Weaviate is so powerful that it can perform an Image Search in an instant. 

So, what is happening under the hood?

Three main steps:
1. Weaviate Database Setup
2. Image Vectorization
3. Querying our Database

#### 1. Weaviate Database Setup
The test data, we have for populating the database contains 5 generated images of people. Each person will have three properties: `filename`, `image`, and `name`. These properties are hence defined in the schema, which is the structure for storing data in Weaviate. Schema is initialized by either running `./import/python/create_schema.py` or `./import/curl/create_schema.sh`.  

Here’s the schema for this application:
```Python
def create_schema(client: weaviate.Client):
  class_obj = {
    "class": "FaceRecognition",
    "moduleConfig": {
        "img2vec-neural": {
            "imageFields": [
                "image"
            ]
        }
    },
    "vectorIndexType": "hnsw",
    "vectorizer": "img2vec-neural",
    "properties": [
      {
        "dataType": [
          "string"
        ],
        "name": "filename"
      },
      {
        "dataType": [
            "blob"
        ],
        "name": "image"
      },
      {
        "dataType": [
          "string"
        ],
        "name": "name"
      }
    ]
  }
```

Note that the Image files are first converted to Base64 encoding, before uploading to the Database. 
#### 2. Image Vectorization
This application uses Weaviate’s img2vec-neural module to vectorize images. This module was trained on the ResNet-50 model to produce meaningful vectorization, which can be used for semantic similarity search downstream. 

#### 3. Querying our Database
After having populated the Database, we need to Query it to effectively retrieve Data. 

We can find the retrieval query in `./frontend/src/weaviate/WeaviateClient.ts`. The main function performing Similarity Search is `findImage()`. 

```TypeScript
  public async findImage(img: string): Promise<PersonImage[]> {
    if (img && img.length > 0) {
      const results = await this.client.graphql
          .get()
          .withClassName('FaceRecognition')
          .withNearImage({image: img})
          .withFields('filename image name _additional{ certainty }')
          .withLimit(1)
          .do()
      return this.toPersonImage(results)
    }
    return []
  }
```

Notice the usage of `withNearImage()`. It uses Weaviate’s `nearImage` search operator, which finds similarity on base64 encoded images. In the `withFields()` method, we can notice an additional field `certainty`. It measures the cosine distance between two vectors (images). The closer the distance, the more similar the images. Note that the query is also formulated so that the response is limited to one result.

## [Prerequisites](https://github.com/weaviate-tutorials/DEMO-face-recognition#prerequisites)

- Docker & Docker-Compose
- Bash
- Node.js and npm/yarn if you also want to run the frontend

## [Setup instructions](https://github.com/weaviate-tutorials/DEMO-face-recognition#setup-instructions)

(TO DO: Add setup instructions)

## [Usage instructions](https://github.com/weaviate-tutorials/DEMO-face-recognition#usage-instructions)

The easiest way to start and stop the demo is to use the `start.sh` and `stop.sh` scripts.

To start the demo issue: `bash start.sh`.

To stop the demo issue: `bash stop.sh`.

Directory `look_for` contains similar images of faces used in the demo (generated by [generated.photos](https://generated.photos/)) that you can use to drag & drop them into the search area to perform similarity searches.

### [How to run with your own images](https://github.com/weaviate-tutorials/DEMO-face-recognition#how-to-run-with-your-own-images)

Simply create a subfolder in `./images` folder. Place your image in subfolder and create `name.txt` file and put a name there prior running the import script. The script looks for `.jpg` file ending, but Weaviate supports other image types as well, you can adopt those if you like.

It is a minimal example using only few images, but you can add any amount of images yourself!

## [Dataset](https://github.com/weaviate-tutorials/DEMO-face-recognition#dataset)

The images used in this demo are generated by [generated.photos](https://generated.photos/). This [terms and conditions](https://generated.photos/terms-and-conditions) page describes under which rights those images can be used.
