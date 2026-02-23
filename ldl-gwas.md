[Link do matrix table](http://149.156.177.112/projects/ifpan-hail-workshop/)


Krótka instrukcja ;)

`matrix_table` zawiera warianty genetyczne dla ok. 80k uczestników. Warianty zostały zannotowane narzędziem VEP, aby przewidzieć, które wariatny są loss-of-function (`lof`). Macierz w wierszach zawiera te informacje wraz w wieloma innymi. 

- `mt.transcript_consequences.lof` - kolumna zawiera pewność predykcji. HC - high confidence, LC - low confidence, lub brak przewidywanego lof.

Każdy wariant może należeć do wielu transkryptów, dlatego warianty często występuję w większej liczbie. Możemy wybrać transkrypt "kanoniczny" - reprezentant dla genu (kolumna: `mte.transcript_consequences.canonical`, 1 oznacza, że dany transkrypt jest kanoniczny)

- `mt.ldl_level` zawiera syntetyczny poziom ldl dla każdego pacjenta

Celem jest zbadanie asocjacji tego fenotypu z wariantami lof. W tym celu należy wybrać warianty 'HC', a następnie zaggregować je w obrębie genu `mt.transcript_consequences.gene_symbol`, tak aby `1` oznaczało, że dana osoba posiada _przynajmniej_ jeden wariant lof w genie.

Przykłady kodu:

filtrowanie
```
mt = mt.filter_rows(
    (mt.transcript_consequences.lof == 'HC')
    & (mt.transcript_consequences.canonical == 1)
).persist()
```

aggregacja
```
gene_mt = (
    mt.group_rows_by(mt.transcript_consequences.gene_symbol)
    .aggregate(
        has_lof=hl.int(
            hl.agg.any(
                (mt.transcript_consequences.lof == 'HC') & mt.GT.is_non_ref()
            )
        )
    )
)
```
