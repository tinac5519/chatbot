# chatbot
for Dialogflow api
需先有Dialogflow的agent，讀到最後一個intent會給QRcode
經過api中轉可將使用者對話記錄至資料庫，供後續分析使用。
```
import firebase_admin
from firebase_admin import credentials
from firebase_admin import firestore
from firebase_admin import db
import uuid

#firestore credentials
cred = credentials.Certificate('firestore_test.json')
firebase_admin.initialize_app(cred, {
    'databaseURL': "your_firebase"
})
count = 0
intent_list = ['Ending1', 'Ending2', 'Ending3', 'Ending4']

def insert_data(request):
  global count
  req = request.get_json()
  res = {"fulfillmentMessages": req["queryResult"]['fulfillmentMessages']}
  session = req['session'][-8:]
  intent = req['queryResult']['intent']['displayName']
  user = req['queryResult']['queryText']
  doc = {intent:user}
  doc_ref=db.reference(session)
  doc_ref.update(doc)
  if intent in intent_list and count<=50:
    uid = uuid.uuid4()
    uu = {'payload': {'richContent': [[{'options': [{'text': '點擊領取限量小禮',"link": "your_web"+str(uid)}], 'type': 'chips'}]]}}
    doc_ref.update({'uid':str(uid)})
    res['fulfillmentMessages'].append(uu)
    count+=1
  elif intent in intent_list and count>50:
    uu = {'text': {'text': ['限量禮物已全數領取完畢了喔！']}}
    res['fulfillmentMessages'].append(uu)
  return res
```
