# Robot Report



Start server on port 8000
```
echo t2
```{{execute T2}}

```
docker exec -it robot bash
cd /home/robotuser/report/
python3 -m http.server 
```{{execute T2}}

## access to web on report-tab or here: 
https://[[HOST_SUBDOMAIN]]-8000-[[KATACODA_HOST]].environments.katacoda.com/report.html
