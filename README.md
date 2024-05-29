# DMPonline

A Python interface to the DMPonline API.
For API documentation, see [https://github.com/DMPRoadmap/roadmap/wiki/API-documentation](https://github.com/DMPRoadmap/roadmap/wiki/API-documentation).

Installation:
```shell
pip install dmponline
```

Usage example:

```python
from dmponline import DMPonline

# initialize DMPonline class
dmp_api = DMPonline(<YOUR-DMPONLINE-API-TOKEN>, token_user=<YOUR-DMPONLINE-USER-EMAIL>)
# retreive pandas dataframe with all DMPs in your organization
dmps = dmp_api.plan_statistics(params={'remove_tests': 'false'})
```

## Command line
The package also ships a command line script to retreive an overview of all questions in a particular DMP.

Usage example:

```shell
question_overview -i DMPONLINE_PLAN_ID -t <YOUR-DMPONLINE-API-TOKEN>
```

## Retrieving structured DMP:s

If your organization has enabled [question id feature for DMPOnline](https://www.youtube.com/watch?v=ihu9Bv8qN-M) for templates you can retrieve structured DMP:s using the following script snippet. Remember to modify the template ID

```python
from src import dmponline
import json


dmp_api = dmponline.DMPonline(url="https://dmptuuli.fi/api/",token="your-dmponline-token")

plans = dmp_api.get_plans(params={'template':'1440979378'})

for plan in plans['plans']:
    for content in plan['plan_content']:
        dmp = {
            'id':plan['id'],
            'title':plan['title'],
            'last_updated':plan['last_updated'],
            'created':plan['creation_date']
            }
        if plan.get("principal_investigator"):
            dmp["principal_investigator"] = plan.get("principal_investigator")
        if plan.get("data_contact"):
            dmp["data_contact"] = plan.get("data_contact")
        if plan.get("users"):
            dmp["users"] = plan.get("users")
        if plan.get("grant_number"):
            dmp["grant_number"] = plan.get("grant_number")
        if plan.get("funder"):
            dmp["funder"] = plan.get("funder")
            
        question_data = {}
        for section in content['sections']:
    
            for question in section['questions']:
                selected = []
                if len(question['question_identifiers'])>0:
                    for identifier in question['question_identifiers']:
                        if identifier['name'] == 'tuni/v1':
                            name = identifier['name'] + "::" + identifier['value']
                            if question.get("answer"):
                                options = question.get("answer").get("options")
                                if options:
                                    for option in options:
                                        selected.append(option.get("answer_identifier"))
                                        question_data[name] = selected
                                else:
                                    answer_text = question.get("answer").get("text")
                                    question_data[name] = answer_text
                                
        dmp["dmp"] = question_data
    if question_data:
        with open(str(plan['id'])+".json", "w") as outfile: 
            json.dump(dmp, outfile,indent=2)
                    


```