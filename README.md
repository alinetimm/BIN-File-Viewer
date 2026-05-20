# Plot BIN — Visualizador de Logs ArduPilot

Ferramenta em Python para leitura, visualização e exportação de logs binários (.BIN) do ArduPilot/Pixhawk. Desenvolvida como apoio à campanha de validação experimental do modelo 6-DOF do Whiteboat (Projeto Hydrone, FURG/UFPel).

## Descrição

O ArduPilot grava dados de sensores em arquivos binários no formato DataFlash (.BIN). Este script lê esses arquivos usando a biblioteca oficial `pymavlink`, extrai os dados de todos os sensores disponíveis, e gera automaticamente:

- **Gráficos** (interativos ou PDF) de atitude, acelerações, giroscópio, barômetro, GPS, vibração, bateria e potência
- **Arquivos CSV** com os dados brutos de cada sensor, prontos para análise no Excel, MATLAB ou outro software
- **Resumo estatístico** no terminal com média, desvio-padrão e range de cada variável

O script é genérico — funciona com qualquer arquivo .BIN do ArduPilot, independentemente do veículo (USV, UAV, rover) ou configuração de sensores. Se um sensor não estiver presente no log, ele é ignorado automaticamente.

## Instalação

### Requisitos

- Python 3.8+
- Bibliotecas: `pymavlink`, `matplotlib`, `numpy`

### Instalar dependências

```bash
pip install pymavlink matplotlib numpy
```

## Uso

### Básico — visualização interativa

```bash
python3 plot_bin.py 00000180.BIN
```

Abre janelas interativas com os gráficos. Você pode dar zoom, pan e salvar individualmente.

### Salvar como PDF

```bash
python3 plot_bin.py 00000180.BIN --save
```

Gera um PDF por grupo de sensores na pasta atual.

### Exportar CSVs

```bash
python3 plot_bin.py 00000180.BIN --csv
```

Gera um CSV por tipo de mensagem (ATT.csv, IMU.csv, BARO.csv, etc.).

### Combinar opções e definir pasta de saída

```bash
python3 plot_bin.py 00000180.BIN --save --csv --outdir resultados/
```

### Sem argumento — seletor interativo

```bash
python3 plot_bin.py
```

Lista os arquivos .BIN na pasta atual e permite selecionar por número.

### Processar múltiplos arquivos (bash)

```bash
for f in *.BIN; do
    python3 plot_bin.py "$f" --save --csv --outdir "resultados_${f%.BIN}"
done
```

## Gráficos gerados

| Arquivo | Conteúdo | Sensor |
|---------|----------|--------|
| `ATT_angles.pdf` | Roll, Pitch, Yaw | ATT |
| `ATT_desired.pdf` | Atitude real vs desejada | ATT |
| `IMU_accel.pdf` | Acelerações X, Y, Z | IMU |
| `IMU_gyro.pdf` | Velocidades angulares X, Y, Z | IMU |
| `BARO_alt.pdf` | Altitude barométrica | BARO |
| `BARO_press_temp.pdf` | Pressão e temperatura (eixo duplo) | BARO |
| `GPS_speed.pdf` | Velocidade GPS | GPS |
| `VIBE.pdf` | Vibrações nos 3 eixos | VIBE |
| `BAT.pdf` | Tensão e corrente (eixo duplo) | BAT |
| `POWR.pdf` | Tensões de alimentação | POWR |

## CSVs exportados

Cada CSV contém uma coluna `time_s` (tempo em segundos desde o início do log) seguida dos campos do sensor. Exemplo do `ATT.csv`:

```
time_s,DesRoll,DesPitch,DesYaw,ErrRP,ErrYaw,Pitch,Roll,Yaw
0.000000,0.000000,0.000000,299.450000,0.230000,0.120000,4.010000,-5.190000,299.450000
0.020000,...
```

## Como adicionar novos sensores

Edite a lista `PLOT_GROUPS` no início do script. Cada grupo segue este formato:

```python
{
    'name': 'Nome do Gráfico',          # título
    'msg_type': 'RCIN',                 # tipo de mensagem ArduPilot
    'fields': ['C1', 'C2', 'C3'],       # campos a plotar
    'labels': ['Canal 1', 'Canal 2', 'Canal 3'],  # legendas
    'ylabel': 'PWM [μs]',              # label do eixo Y
    'filename': 'RCIN_channels',        # nome do PDF
},
```

Para descobrir quais tipos de mensagem e campos estão disponíveis em um log, rode o script uma vez — o resumo no terminal lista todos os tipos encontrados.

## Tipos de mensagem comuns do ArduPilot

| Tipo | Descrição |
|------|-----------|
| `ATT` | Ângulos de Euler (Roll, Pitch, Yaw) |
| `IMU` | Acelerômetro e giroscópio |
| `BARO` | Barômetro (altitude, pressão, temperatura) |
| `GPS` | Posição e velocidade GPS |
| `MAG` | Magnetômetro |
| `BAT` | Bateria (tensão, corrente) |
| `RCIN` | Sinais de entrada do rádio controle |
| `RCOU` | Sinais de saída para os motores |
| `VIBE` | Níveis de vibração |
| `POS` | Posição estimada pelo EKF |
| `XKF1-5` | Estados internos do filtro de Kalman estendido |
| `PARM` | Parâmetros de configuração |

## Confiabilidade

A leitura dos dados é feita pela biblioteca `pymavlink`, que é a biblioteca oficial do projeto ArduPilot. É a mesma usada internamente pelo Mission Planner, QGroundControl e UAV Log Viewer. Os dados extraídos são os valores brutos gravados pela Pixhawk, sem nenhum processamento, interpolação ou filtragem adicional.

## Contexto

Esta ferramenta foi desenvolvida como parte da campanha de validação experimental do modelo 6-DOF do catamarã Whiteboat, plataforma colaborativa para o HUAUV Hydrone Jv2. Os logs processados por este script alimentam a comparação simulação vs experimental apresentada em:

- Timm, A. G. G. (2026). *Modelagem 6-DOF de um veículo de superfície não tripulado para missões colaborativas com veículos híbridos aéreo-subaquáticos não tripulados.* Dissertação de Mestrado, UFPel.
- Timm, A. G. G., Barbosa, K. D. R. C., Drews-Jr., P. L. J. (2026). *6-DOF Dynamic Modeling and Experimental Validation of a Catamaran USV for Collaborative Missions with Hybrid Aerial-Underwater Vehicles.* (em preparação)

## Licença

Uso livre para fins acadêmicos e de pesquisa.
