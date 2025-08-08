# Automatisation des Jobs Slurm sur Jean-Zay

Automatisez la soumission, le suivi et l'archivage de vos jobs `sbatch` sur le supercalculateur **Jean-Zay** du CNRS. Ce projet propose un script Bash simple pour lancer plusieurs jobs en parallèle, suivre leur progression et archiver les logs correspondants.

## Sommaire
- [Caractéristiques](#caractéristiques)
- [Prérequis](#prérequis)
- [Utilisation](#utilisation)
- [Personnalisation](#personnalisation)
- [Licence](#licence)

## Caractéristiques
- Soumission simultanée de plusieurs scripts Slurm.
- Récupération et stockage automatique des IDs de jobs.
- Boucle de surveillance pour suivre l'avancement des jobs avec `squeue`.
- Archivage des fichiers `.out` et `.err` une fois les jobs terminés.

## Prérequis
- Accès au cluster Jean-Zay et à la commande `sbatch`.
- Des scripts Slurm configurés avec les options `--output` et `--error`.
- Bash 4+

## Utilisation
1. Placez vos scripts Slurm (ex: `myjob1.slurm`, `myjob2.slurm`, `myjob3.slurm`) dans le même dossier que le script ci-dessous.
2. Adaptez éventuellement les chemins de logs et les options dans le script.

```bash
#!/usr/bin/env bash

# 1. Soumission des jobs en parallèle
LOGDIR="$HOME/logs"
JOBLIST="$LOGDIR/job_ids.txt"
mkdir -p "$LOGDIR"
: > "$JOBLIST"

for script in myjob1.slurm myjob2.slurm myjob3.slurm; do
    output=$(sbatch "$script")
    job_id=$(echo "$output" | awk '{print $4}')
    echo "$job_id $script" >> "$JOBLIST"
    echo "Job $job_id soumis pour $script"
done

# 2. Suivi des jobs
while true; do
    running=""
    while read -r job_id script; do
        if squeue -j "$job_id" >/dev/null 2>&1; then
            running="$running $job_id"
        fi
    done < "$JOBLIST"

    if [[ -z $running ]]; then
        echo "Tous les jobs sont terminés."
        break
    else
        echo "Jobs encore en cours : $running"
        sleep 30
    fi
done

# 3. Archivage des logs
ARCHIVE="$LOGDIR/archive_$(date +%Y%m%d_%H%M)"
mkdir -p "$ARCHIVE"

while read -r job_id script; do
    base=$(basename "$script" .slurm)
    mv "${base}.${job_id}.out" "$ARCHIVE" 2>/dev/null
    mv "${base}.${job_id}.err" "$ARCHIVE" 2>/dev/null
done < "$JOBLIST"

echo "Logs archivés dans $ARCHIVE"
```

## Personnalisation
- Ajustez l'intervalle de vérification dans la boucle `while` (30 s par défaut).
- Remplacez `squeue` par `sacct` pour plus d'informations sur les jobs terminés.
- Analysez automatiquement les fichiers `.err` pour relancer les jobs en échec.

## Licence
Ce projet est sous licence [Apache 2.0](LICENSE).

