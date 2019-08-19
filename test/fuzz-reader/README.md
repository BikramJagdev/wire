## reader fuzzing

Fuzzing is a technique for sending arbitrary input into functions to see what happens. Typically this is done to weed out crashes/panics from software, or to detect bugs. In some cases all possible inputs are ran into the program (i.e. if a function accepts 16-bit integers it's trivial to try them all).

In this directory we are fuzzing `reader.go` from the root level -- In specific the `Reader.Read()` method.

### Running

If you need to setup `go-fuzz`, run `make install`.

Otherwise, run `make` and watch the output. Fix any crashes that happen, thanks!

See the `go-fuzz` project for more docs: https://github.com/dvyukov/go-fuzz

### Corpus

Right now our corpus exists mostly of test files. As a machine runs go-fuzz files are written to the `corpus/` directory.

### Downloading crashers from Kubernetes cluster

If the [`moov/wirefuzz`](https://hub.docker.com/r/moov/wirefuzz) Docker image is running in a Kubernetes cluster you can download the crashers from a mounted volume by executing the following commands.

```
# Get current pod name
$ kubectl get pods -n apps | grep wirefuzz
wirefuzz-6bbdc574f5-pl2zm        1/1       Running     0          1h
```

Then using the [volume's mount path](https://github.com/moov-io/infra/blob/master/lib/apps/18-wirefuzz.yml) select any crasher files.

```
$ kubectl exec -n apps wirefuzz-6bbdc574f5-pl2zm -- ls -la /go/src/github.com/moov-io/wire/test/fuzz-reader/crashers/
total 28
drwxr-xr-x    3 root     root          4096 Jan 30 00:26 .
drwxr-xr-x    1 root     root          4096 Jan 14 17:30 ..
# Download files, replace <file> with a crasher file
$ kubectl cp 'apps/wirefuzz-6bbdc574f5-pl2zm:/go/src/github.com/moov-io/wire/test/fuzz-reader/crashers/<file>' ./
```

### fuzzit integration

[fuzzit](https://fuzzit.dev/) is a free SaaS for automated fuzzing. They offer free fuzzing for OSS projects so we've setup wirefuzz for their service. After creating a target in the web UI we copied our corpus up (`tar cf wirefuzz.tar *.txt` in `test/fuzz-reader/corpus/` then `gzip wirefuzz.tar`).

We need to then copy down their bash script (`fuzzit completion > fuzzit.sh && chmod +x ./fuzzit.sh`) and create our job:

```
# In test/fuzz-reader/
$ fuzzit create job --type=fuzzing wirefuzz fuzzit.sh
2019/08/19 10:50:59 Creating job...
2019/08/19 10:50:59 Uploading fuzzer...
2019/08/19 10:51:05 Starting job
2019/08/19 10:51:05 Job baAZGS3OQfeCEL1D6HtL started succesfully
2019/08/19 10:51:05 Job created successfully
```
