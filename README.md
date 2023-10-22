##  Primer of a string processing pipeline on Triton Inference Server on a CPU-only Docker-less system

Here we have a few example Python models accepting a batch of JSON objects and returning a batch of JSON objects. These models are connected in a sequential **"ensemble"** pipeline (in Triton Inference Server lingo).

If you can't use the provided Docker images of `trintonserver`, please compile the `tritonserver` binary with Python backend [from source](https://github.com/triton-inference-server/server/blob/main/docs/customization_guide/build.md#cpu-only-build). See [buildtritoninferenceserver.yml](./.github/workflows/buildtritoninferenceserver.yml) for steps or use the GitHub Action / uploaded artifacts (for Ubuntu 22.04).

```shell
#place tritonserver into /opt/tritonserver
#sudo mkdir -p /opt/ && sudo ln -s "$PWD/build/install/tritonserver" /opt
#export PATH="/opt/tritonserver/bin/:$PATH"

tritonserver --model-repository "$PWD/models" --log-verbose=1

curl -i http://localhost:8000/v2/health/ready
# HTTP/1.1 200 OK

curl -i -X POST localhost:8000/v2/models/modelA/infer -H 'Inference-Header-Content-Length: 140' -H "Content-Type: application/octet-stream" --data-binary '{"inputs":[{"name":"INPUT0","shape":[15],"datatype":"UINT8","parameters":{"binary_data_size":15}}],"parameters":{"binary_data_output":true}}{"hi": "hello"}'
# HTTP/1.1 200 OK
#Content-Type: application/octet-stream
#Inference-Header-Content-Length: 143
#Content-Length: 171
#{"model_name":"modelA","model_version":"1","outputs":[{"name":"OUTPUT0","datatype":"UINT8","shape":[28],"parameters":{"binary_data_size":28}}]}{"hi": "modelAmodelA:hello"}

curl -i -X POST localhost:8000/v2/models/modelB/infer -H 'Inference-Header-Content-Length: 140' -H "Content-Type: application/octet-stream" --data-binary '{"inputs":[{"name":"INPUT0","shape":[15],"datatype":"UINT8","parameters":{"binary_data_size":15}}],"parameters":{"binary_data_output":true}}{"hi": "hello"}'
# HTTP/1.1 200 OK
#Content-Type: application/octet-stream
#Inference-Header-Content-Length: 143
#Content-Length: 171
#{"model_name":"modelB","model_version":"1","outputs":[{"name":"OUTPUT0","datatype":"UINT8","shape":[28],"parameters":{"binary_data_size":28}}]}{"hi": "modelBmodelB:hello"}

curl -i -X POST localhost:8000/v2/models/pipeline/infer -H 'Inference-Header-Content-Length: 140' -H "Content-Type: application/octet-stream" --data-binary '{"inputs":[{"name":"INPUT0","shape":[15],"datatype":"UINT8","parameters":{"binary_data_size":15}}],"parameters":{"binary_data_output":true}}{"hi": "hello"}'
#HTTP/1.1 200 OK
#Content-Type: application/octet-stream
#Inference-Header-Content-Length: 220
#Content-Length: 261
#{"model_name":"pipeline","model_version":"1","parameters":{"sequence_id":0,"sequence_start":false,"sequence_end":false},"outputs":[{"name":"OUTPUT0","datatype":"UINT8","shape":[41],"parameters":{"binary_data_size":41}}]}{"hi": "modelBmodelB:modelAmodelA:hello"}

# the requests above can also be generated by
python3 client.py modelA hello1
python3 client.py modelB hello2
python3 client.py pipeline hello3
```

## References
- https://github.com/triton-inference-server/server/blob/main/docs/protocol/extension_generate.md
- https://github.com/triton-inference-server/python_backend
- https://github.com/triton-inference-server/python_backend/tree/main/examples/preprocessing
- https://github.com/triton-inference-server/python_backend/tree/main/examples/auto_complete
- https://blog.ml6.eu/triton-ensemble-model-for-deploying-transformers-into-production-c0f727c012e3
