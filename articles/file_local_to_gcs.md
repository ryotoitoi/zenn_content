---
title: "Google Cloud Strage(GCS)にcsvをアップロードしてみる。(python)"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['Google Cloud', 'GCS', 'Google Cloud Strage']
published: false
---

# やり方
まず、
Google Cloud Console > IAMと管理 > サービスアカウント > キー
から鍵を作成する。
今回はJSONで作成。


'''python
from gcloud import storage
from oauth2client.service_account import ServiceAccountCredentials
import json

CREDENTIAL_PATH = 'YOUR_CREDENTIAL.json' #GCS CREDENTIALのパス
BUCKETS_NAME = 'test2022116' #GCSのバケット名
DESTINATION_BLOB_NAME = 'testgcs.csv' #GCSでの命名
SOURCE_FILE_NAME = 'test.csv' #GCSにアップロードしたいファイルのパス

with open(CREDENTIAL_PATH) as f:
    credentials_dict = json.load(f)

credentials = ServiceAccountCredentials.from_json_keyfile_dict(
    credentials_dict
)

client = storage.Client(credentials=credentials, project='myproject')
bucket = client.get_bucket(BUCKETS_NAME)

blob = bucket.blob(DESTINATION_BLOB_NAME)
blob.upload_from_filename(SOURCE_FILE_NAME)
'''

