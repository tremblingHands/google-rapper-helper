
# helper:google/rappor 仿真

## 默认运行参数

tests/regtest_spec.py 
23

```
    ('demo3 exp     100 100000 10', '32 1 64', '0.25 0.75 0.5', '100 v[0-9]*9$'),
```



## 获取value:Freq

tests/compare_dist.R
149

```r
  a <- data.frame(index = actual_values,
                  # Calculate the true proportion
                  proportion = actual$count / total,
                  dist = "actual")

  r <- data.frame(index = rappor_values,
                  proportion = rappor$proportion,
                  dist = rep("rappor", length(rappor_values)))
```

131

```r
  actual <- ctx$actual  # from the ground truth file
  rappor <- ctx$rappor  # from output of AnalyzeRAPPOR
```

251

```r
  ctx <- LoadContext(input_case_prefix)
  ctx$rappor <- RunRappor(input_case_prefix, input_instance_prefix, ctx)
  ctx$actual <- LoadActual(input_instance_prefix)
```

### 输出value:Freq

258

```
  #omni
  filename <- file.path(output_dir, 'omni.csv')
  write.csv(ctx$actual, file = filename, row.names = FALSE)
```

## 生成map 

regtest.sh
139

```bash
  local true_map_path=$case_dir/case_true_map.csv

  bin/hash_candidates.py \
    $params_path \
    < $case_dir/case_unique_values.txt \
    > $true_map_path
```

153

```
  bin/hash_candidates.py \
    $params_path \
    < $case_dir/case_candidates.txt \
    > $case_dir/case_map.csv
```

## 替换truedata.csv

### 将truedata.csv放置在rapper目录下，修改regtest.sh

regtest.sh ：42

```
# omni
TRUE_VALUES_PATH=omni_truedata.csv
```


## 分析

### demo.sh:46

```bash
quick-python() {  
  ./regtest.sh run-seq '^demo3' python
}
```

### regtest.sh:381

```bash
run-seq() {
  local spec_regex=${1:-'^r-'}  # grep -E format on the spec
  shift

  time _run-tests $REGTEST_SPEC $spec_regex F $@
}
```

```
`REGTEST_SPEC=tests/regtest_spec.py`
`spec_regex='^demo3'`
`$@=python`
```

### regtest.sh:329

_run-tests()

```
spec_gen=tests/regtest_spec.py
spec_regex=^demo3
parallel=F
impl=python
instances=1
46:REGTEST_BASE_DIR=_tmp
regtest_dir=_tmp/python
cases_list=_tmp/python/test-cases.txt
```

vim test-cases.txt

```
demo3 exp     100 100000 10 32 1 64 0.25 0.75 0.5 100 v[0-9]*9$
```


```bash
func=_run-one-instance  # output to the console
processors=1
```

360

```bash
cat _tmp/python/test-cases.txt     | xargs -l -P 1 -- ./regtest.sh _setup-one-case python     || test-error
```

367

```bash
_setup-test-instances 1 python < _tmp/python/test-cases.txt > _tmp/python/test-instances.txt
```

369

```bash
cat _tmp/python/test-instances.txt     | xargs -l -P 1 -- ./regtest.sh _run-one-instance || test-error
```

374

```bash
make-summary _tmp/python python
```

_run-one-instance():

179:

```
tests/gen_true_values.R exp 100 100000                             10 64                             _tmp/python/demo3/1/case_true_values.csv
```

198

```
tests/rappor_sim.py  --num-bits 32  --num-hashes 1  --num-cohorts 64  -p 0.25  -q 0.75         -f 0.5  < _tmp/python/demo3/1/case_true_values.csv  > _tmp/python/demo3/1/case_reports.csv
```
