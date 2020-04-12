# Notes

My notes on testing using Stefan's scripts.

## Builder

Already done by Stefan. Only two additions:

- On Windows try `docker run --detach --hostname gitlab.example.com --publish 443:443 --publish 80:80 
--publish 22:22 --name gitlab-11.2 --restart always --volume glconfig:/etc/gitlab --volume gllogs:/var/log/gitlab --volume gldata:/var/opt/gitlab gitlab/gitlab-ce:11.2.0-ce.0`.
- You need to wait 5-10 minutes for the docker image to boot.

## Runner

Tried in the clojure repl:

```
Clojure 1.10.0
user=> (load-file "src/openapi_pbt/gitlab_experiments.clj")       
nil
user=>
```

This did something, not sure what, but didn't produce any discernable results.  However, reading the source, it looks like it's similar to `schemathesis` - there are various [OpenAPI files](./ICST-2020/gitlab-files/) and you run tests using them.

I tried using the files as-is but some of them were not valid swagger schemas. So I did a little cleanup using swagger hub, converting to OpenAPI 3.0 along the way.

Here is the result of `gitlab-1.yaml`:

```bash
schemathesis run --base-url http://localhost --header "Private-Token: LNASRzMc6mGFybm4LFNo" .\gitlab-1.yaml
================ Schemathesis test session starts ================
platform Windows -- Python 3.7.3, schemathesis-1.1.0, hypothesis-5.8.1, hypothesis_jsonschema-0.12.0, jsonschema-3.2.0
rootdir: C:\Users\MikeSolomon\devel\test-apis\gitlab\ICST-2020\gitlab-files
hypothesis profile 'default' -> database=DirectoryBasedExampleDatabase('C:\\Users\\MikeSolomon\\devel\\test-apis\\gitlab\\ICST-2020\\gitlab-files\\.hypothesis\\examples')
Schema location: file:///C:/Users/MikeSolomon/devel/test-apis/gitlab/ICST-2020/gitlab-files/gitlab-1.yaml
Base URL: http://localhost
Specification version: Open API 3.0.0
Workers: 1
collected endpoints: 1

GET /api/v4/groups .                                       [100%] 

============================ SUMMARY =============================

not_a_server_error            100 / 100 passed          PASSED    

======================= 1 passed in 3.84s ========================
```

No bugs found...

Here is the result of gitlab-2.yaml

```bash
schemathesis run --base-url http://localhost --header "Private-Token: LNASRzMc6mGFybm4LFNo" .\gitlab-1.yaml
================ Schemathesis test session starts ================
platform Windows -- Python 3.7.3, schemathesis-1.1.0, hypothesis-5.8.1, hypothesis_jsonschema-0.12.0, jsonschema-3.2.0
rootdir: C:\Users\MikeSolomon\devel\test-apis\gitlab\ICST-2020\gitlab-files
hypothesis profile 'default' -> database=DirectoryBasedExampleDatabase('C:\\Users\\MikeSolomon\\devel\\test-apis\\gitlab\\ICST-2020\\gitlab-files\\.hypothesis\\examples')
Schema location: file:///C:/Users/MikeSolomon/devel/test-apis/gitlab/ICST-2020/gitlab-files/gitlab-1.yaml
(.venv) PS C:\Users\MikeSolomon\devel\test-apis\gitlab\ICST-2020\gitlab-files> schemathesis run --base-url http://localhost --header "Private-Token: LNASRzMc6mGFybm4LFNo" .\gitlab-2.yaml
================ Schemathesis test session starts ================
platform Windows -- Python 3.7.3, schemathesis-1.1.0, hypothesis-5.8.1, hypothesis_jsonschema-0.12.0, jsonschema-3.2.0
rootdir: C:\Users\MikeSolomon\devel\test-apis\gitlab\ICST-2020\gitlab-files
hypothesis profile 'default' -> database=DirectoryBasedExampleDatabase('C:\\Users\\MikeSolomon\\devel\\test-apis\\gitlab\\ICST-2020\\gitlab-files\\.hypothesis\\examples')
Schema location: file:///C:/Users/MikeSolomon/devel/test-apis/gitlab/ICST-2020/gitlab-files/gitlab-2.yaml
Base URL: http://localhost
Specification version: Open API 3.0.0
Workers: 1
collected endpoints: 1

GET /api/v4/groups F                                       [100%]

============================ FAILURES ============================
______________________ GET: /api/v4/groups _______________________
1. Received a response with 5xx status code: 500

Check           : not_a_server_error
Query           : {'page': 461168601842738792}

Run this Python code to reproduce this failure: 

    requests.get('http://localhost/api/v4/groups', params={'page': 461168601842738792})

Or add this option to your command line parameters: --hypothesis-seed=283564230034527281420651305753371641243
============================ SUMMARY =============================

not_a_server_error            169 / 272 passed          FAILED    

======================= 1 failed in 12.05s =======================
```

Let's verify:

```python
requests.get('http://localhost/api/v4/groups', params={'page': 461168601842738792})
```

Yup! Internal server error, likely because of the ridiculously large page value.

## Mocker

No mocking needed.