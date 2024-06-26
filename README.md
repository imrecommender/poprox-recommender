# poprox-recommender

This repository contains the POPROX recommender code — the end-to-end logic for
producing recommendations using article data, user histories and profiles, and
trained models.

- [Installation](#installation)
- [Local Development](#localdevelopment)
- [License](#license)

## Installation

Model and data files are managed using [dvc][].  The `conda-lock.yml` provides a
[conda-lock][] lockfile for reproducibly creating an environment with all
necessary dependencies.

[dvc]: https://dvc.org
[conda-lock]: https://conda.github.io/conda-lock/

To set up the environment with Conda:

```
conda install -n base -c conda-forge conda-lock
conda lock install -n poprox-recsys
conda activate poprox-recsys
```

If you use `micromamba` instead of a full Conda installation, it can directly use the lockfile:

```
micromamba create -n poprox-recs -f conda-lock.yml
```

To get the data and models, there are two steps:

1.  Obtain the credentials for the S3 bucket and put them in `.env` (the environment variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`)
2.  `dvc pull`


## Local Development

There are two sets of dependencies. To install the Serverless framework and Node dependencies:

```console
npm install -g serverless
npm install
```

To install Python dependencies:

```console
pip install -r requirements.txt
```

To run the API endpoint locally:

```console
serverless offline start --reloadHandler
```

Once the local server is running, you can send requests to `localhost:3000`. A request with this JSON body:

```json
{
    "past_articles": [
        {
            "article_id": "e7605f12-a37a-4326-bf3c-3f9b72d0738d",
            "title": "title 1",
            "content": "content 1",
            "url": "url 1"
        }
    ],
    "todays_articles": [
        {
            "article_id": "7e5e0f12-d563-4a60-b90a-1737839389ff",
            "title": "title 2",
            "content": "content 2",
            "url": "url 2"
        }
    ],
    "click_data": [
        {
            "account_id": "977a3c88-937a-46fb-bbfe-94dc5dcb68c8",
            "article_ids": [
                "e7605f12-a37a-4326-bf3c-3f9b72d0738d"
            ]
        }
    ],
    "num_recs": 1
}
```

should receive this response:

```json
{
    "recommendations": {
        "977a3c88-937a-46fb-bbfe-94dc5dcb68c8": [
            {
                "article_id": "7e5e0f12-d563-4a60-b90a-1737839389ff",
                "title": "title 2",
                "content": "content 2",
                "url": "url 2",
                "published_at": "1970-01-01T00:00:00Z",
                "mentions": []
            }
        ]
    }
}
```

You can test this by sending a request with curl:

```console
$ curl -X POST -H "Content-Type: application/json" -d @tests/basic-request.json localhost:3000

{"recommendations": {"977a3c88-937a-46fb-bbfe-94dc5dcb68c8": [{"article_id": "7e5e0f12-d563-4a60-b90a-1737839389ff", "title": "title 2", "content": "content 2", "url": "url 2", "published_at": "1970-01-01T00:00:00Z", "mentions": []}]}}
```
