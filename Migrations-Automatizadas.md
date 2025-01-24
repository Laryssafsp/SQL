Criar um Script para unir arquivos de inserções, como por exemplo, para serem utilizados em uma única vez - automarizado

- criar o arquivo: to_migration.ps1 [win]
- criar o arquivo: to_migration.sh [linux]
    
```ps1
#Pegar o diretório atual
$scriptDirectory = Split-Path -Path $MyInvocation.MyCommand.
Definition -Parent

#Arquivo saida com todos sql
$outputFile = Join-Path -Path $scriptDirectory -ChildPath
"migration.sql"


#Verifica se arquivo ja existe, se existir deleta
if (Test-Path $outputFile) {
    Remove-Item $outputFile
}

#Pega Conteúdo dos arquivos
$sqlFiles = Get-ChildItem -Path $scriptDirectory -Filter *. sql
-File | Sort-Object Name

#Concatena Arquivos
foreach($file in $sqlFiles) {
  Get-Content $file.FullName | Out-File -Append -FilePath $outputFile
  "GO" | Out-File -Append -FilePath $outputFile
}

#Escrever mensagem
Write-Host "Todos Arquivos foram combinados em $outputFile


```

para rodar: coloca no terminal o caminho do arquivo
