# Πρακτικός οδηγός για την εξαμηνιαία εργασία

Ο οδηγός αυτός εστιάζει στα πρακτικά θέματα εκτέλεσης και αξιολόγησης επίδοσης που επηρεάζουν άμεσα τα ζητούμενα του εξαμηνιαίου project.

Υποθέτει ότι έχουν ήδη ολοκληρωθεί:

- [04_remote-spark-kubernetes](../04_remote-spark-kubernetes/README.md)
- [05_cluster-queries-rdd-df-sql](../05_cluster-queries-rdd-df-sql/README.md)

## Τι καλύπτει αυτός ο οδηγός

1. Πώς αλλάζουν οι ρυθμίσεις `spark-submit` για executors, cores και memory.
2. Πώς γίνεται σωστή μέτρηση χρόνου εκτέλεσης για δίκαιες συγκρίσεις.
3. Πώς ελέγχουμε και επηρεάζουμε join strategies (`BROADCAST`, `MERGE`, `SHUFFLE_HASH`, `SHUFFLE_REPLICATE_NL`).
4. Πώς οργανώνουμε αναπαραγώγιμα πειράματα για την αναφορά.

## Κράτα τις εκτελέσεις αναπαραγώγιμες

Πριν από κάθε benchmarking session:

- ίδιο dataset και ίδιο code revision
- μία αλλαγή κάθε φορά
- όσο γίνεται σταθερό φορτίο στο cluster
- πολλαπλές επαναλήψεις ανά case (τουλάχιστον 3)
- αναφορά median χρόνου, όχι μόνο μίας μέτρησης

Ξεκίνα από καθαρό shell:

```bash
deactivate 2>/dev/null || true
source ~/bigdata-env.sh
hash -r
command -v spark-submit
```

## 1. Αλλαγή ρυθμίσεων executors

Στην εκφώνηση ζητούνται συγκεκριμένα configurations.  
Η πιο ασφαλής πρακτική είναι `--conf` ανά εκτέλεση (όχι μόνιμη αλλαγή default αρχείων).

### Override ανά εκτέλεση (προτεινόμενο)

```bash
spark-submit \
  --conf spark.executor.instances=4 \
  --conf spark.executor.cores=1 \
  --conf spark.executor.memory=2g \
  hdfs://hdfs-namenode.default.svc.cluster.local:9000/user/$USER/code/<your_script>.py \
  --base-path hdfs://hdfs-namenode.default.svc.cluster.local:9000/user/$USER
```



## 2. Σωστή μέτρηση χρόνου εκτέλεσης


Μη θεωρείς ως αξιόπιστο runtime το shell timing γύρω από το `spark-submit`.  
Σε cluster mode, το `spark-submit` μπορεί να ολοκληρώσει το submission flow ενώ το application συνεχίζει στο Kubernetes.

### Προτεινόμενη μέθοδος Α (κύρια): μέτρηση μέσα στον Python κώδικα

Για συγκρίσεις στην εργασία, προτίμησε timing με `perf_counter()` γύρω από το τελικό Spark action.

```python
from time import perf_counter

start = perf_counter()
result_df.write.mode("overwrite").csv(output_path)  # ή άλλο τελικό action
elapsed = perf_counter() - start
print(f"QUERY_ELAPSED_SECONDS={elapsed:.3f}")
```

Αν ακολουθήσεις αυτή τη μέθοδο, κράτησε ακριβώς το ίδιο timing boundary σε όλες τις υλοποιήσεις (RDD / DF / SQL).

### Προτεινόμενη μέθοδος Β (validation): μέτρηση application duration από το Spark

Χρησιμοποίησε runtime στοιχεία του ίδιου του Spark (Spark UI / History Server), όχι terminal wall-clock.

1. Κάνε submit με μοναδικό app name.
2. Περίμενε να ολοκληρωθεί στο cluster.
3. Κατέγραψε `Application ID` και `Duration` από τον History Server, αν είναι διαθέσιμος.

Παράδειγμα submit:

```bash
spark-submit \
  --conf spark.app.name=Q2_DF_run1_4x1c_2g \
  --conf spark.executor.instances=4 \
  --conf spark.executor.cores=1 \
  --conf spark.executor.memory=2g \
  hdfs://hdfs-namenode.default.svc.cluster.local:9000/user/$USER/code/Q2_df.py \
  --base-path hdfs://hdfs-namenode.default.svc.cluster.local:9000/user/$USER
```

Μετά πάρε τον χρόνο από το πεδίο `Duration` στον History Server, αν είναι διαθέσιμος, για τον πίνακα συγκρίσεων.


## 3. Join strategies: έλεγχος και πειραματισμός

### Δες τι διάλεξε ο Catalyst

Για DataFrame:

```python
result_df.explain("formatted")
```

Για SQL:

```python
spark.sql("<your query>").explain("formatted")
```

Στο physical plan δες κόμβους όπως:

- `BroadcastHashJoin`
- `SortMergeJoin`
- `ShuffledHashJoin`
- `BroadcastNestedLoopJoin` (συχνά σε non-equi ή cross joins)

### Επίβαλε στρατηγική με hints

DataFrame API:

```python
from pyspark.sql.functions import broadcast

joined = left.join(broadcast(right), "zip_code")
# ή:
joined = left.hint("merge").join(right, "zip_code")
joined = left.hint("shuffle_hash").join(right, "zip_code")
joined = left.hint("shuffle_replicate_nl").join(right, "zip_code")
```

Spark SQL:

```sql
SELECT /*+ BROADCAST(dim) */ ...
SELECT /*+ MERGE(a, b) */ ...
SELECT /*+ SHUFFLE_HASH(a) */ ...
SELECT /*+ SHUFFLE_REPLICATE_NL(dim) */ ...
```

## 4. Τελικό checklist πριν την υποβολή

- Υλοποιήθηκαν όλες οι API εκδοχές που ζητά η εκφώνηση.
- Υπάρχει evidence εκτέλεσης για κάθε ζητούμενο resource configuration.
- Υπάρχουν πίνακες σύγκρισης χρόνων και σύντομος τεχνικός σχολιασμός.
- Το repository έχει σταθερό commit για την προφορική εξέταση.
