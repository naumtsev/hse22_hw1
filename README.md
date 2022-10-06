Создаем директорию для перовго ДЗ


```python
mkdir hw1
cd hw1
```

Создаем символические ссылки на данные


```python
ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
```

Выбираем случайно 5 миллионов чтений типа paired-end и 1.5 миллиона чтений типа mate-pairs


```python
seqtk sample -s424 oil_R1.fastq 5000000 > oil-pair1.fastq
seqtk sample -s424 oil_R2.fastq 5000000 > oil-pair2.fastq
seqtk sample -s424 oilMP_S4_L001_R1_001.fastq 1500000 > oil-mate1.fastq
seqtk sample -s424 oilMP_S4_L001_R2_001.fastq 1500000 > oil-mate2.fastq
```

Создаем директорию для результатов


```python
mkdir result1
```

Оцениваем качество исходных чтений и получаем по ним общую статистику


```python
fastqc oil-pair*  oil-mate* -o result1
```


```python
cd result1
multiqc .
cd ..
```

![](/img/rep_1_1.png)
![](/img/rep_1_2.png)
![](/img/rep_1_3.png)

С помощью утилит platanus_trim и platanus_internal_trim подрежим чтения по качеству и удалим адаптеры


```python
platanus_trim  oil-pair1.fastq oil-pair2.fastq
```


```python
platanus_internal_trim oil-mate1.fastq oil-mate2.fastq
```


```python
Оценим качество еще раз
```


```python
mkdir result2
fastqc oil-pair1.fastq.trimmed  oil-pair2.fastq.trimmed oil-mate1.fastq.int_trimmed oil-mate2.fastq.int_trimmed -o result2
cd result2
multiqc .
```

![](/img/rep_2_1.png)
![](/img/rep_2_2.png)
![](/img/rep_2_3.png)

Удаляем ненужные в дальнейшем файлы


```python
rm oil-pair1.fastq oil-pair2.fastq oil-mate1.fastq oil-mate2.fastq
```

Cобираем контиги из подрезанных чтений


```python
platanus assemble -f oil-pair1.fastq.trimmed oil-pair1.fastq.trimmed 
```


```python
Получили out_contig.fa
```

Анализ полученных контигов:

```python
{
    'contig_count': 1469,
    'max_contig': 'seq374_len58306_cov224',
    'max_contig_size': 58306,
    'N50': 'seq393_len11527_cov224',
    'N50_size': 11527
 }
```


Собираем скаффолды из контигов, а также из подрезанных чтений


```python
platanus scaffold -c out_contig.fa -IP1 oil-pair1.fastq.trimmed oil-pair2.fastq.trimmed -OP1 oil-mate1.fastq.int_trimmed oil-mate2.fastq.int_trimmed
```


```python
Получили out_scaffold.fa
```

Анализ полученных скаффолдов:

```python
{
    'scaffold_count': 130,
    'max_scaffold': 'scaffold4_len383647_cov236',
    'max_scaffold_size': 383647,
    'N50': 'scaffold70_len173419_cov224',
    'N50_size': 173419
}
```

Анализ самого большого скаффолда

```python
{
    'sum_gap_size': 103, 'gap_count': 5
}

```

Уменьшаем кол-во гэпов с помощью подрезанных чтений


```python
platanus gap_close -c out_scaffold.fa -IP1 oil-pair1.fastq.trimmed oil-pair2.fastq.trimmed -OP1 oil-mate1.fastq.int_trimmed oil-mate2.fastq.int_trimmed
```


```python
Получили файл out_gapClosed.fa
```


Анализ полученных скаффолдов:

```python
{
    'scaffold_count': 130,
    'max_scaffold': 'scaffold4_cov236',
    'max_scaffold_size': 383574,
    'N50': 'scaffold70_cov224',
    'N50_size': 173397
 }
```

Анализ самого большого скаффолда

```python
{
    'sum_gap_size': 0, 'gap_count': 0
}
```

