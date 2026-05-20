import numpy as np
import matplotlib.pyplot as plt
import sys
import os
import argparse
from collections import defaultdict


PLOT_STYLE = {
    'font.size': 11,
    'font.family': 'DejaVu Sans',
    'axes.grid': True,
    'grid.linestyle': ':',
    'grid.alpha': 0.5,
    'figure.figsize': (14, 8),
}

COLORS = ['#2166ac', '#d62728', '#2ca02c', '#ff7f0e', '#9467bd', '#8c564b']



PLOT_GROUPS = [
    {
        'name': 'Atitude (Roll, Pitch, Yaw)',
        'msg_type': 'ATT',
        'fields': ['Roll', 'Pitch', 'Yaw'],
        'labels': [r'Roll ($\phi$)', r'Pitch ($\theta$)', r'Yaw ($\psi$)'],
        'ylabel': 'Ângulo [graus]',
        'filename': 'ATT_angles',
    },
    {
        'name': 'Atitude Desejada vs Real',
        'msg_type': 'ATT',
        'fields': ['Roll', 'DesRoll', 'Pitch', 'DesPitch'],
        'labels': ['Roll', 'DesRoll', 'Pitch', 'DesPitch'],
        'ylabel': 'Ângulo [graus]',
        'filename': 'ATT_desired',
    },
    {
        'name': 'IMU — Acelerações',
        'msg_type': 'IMU',
        'fields': ['AccX', 'AccY', 'AccZ'],
        'labels': ['AccX (surge)', 'AccY (sway)', 'AccZ (heave)'],
        'ylabel': 'Aceleração [m/s²]',
        'filename': 'IMU_accel',
    },
    {
        'name': 'IMU — Giroscópio',
        'msg_type': 'IMU',
        'fields': ['GyrX', 'GyrY', 'GyrZ'],
        'labels': ['GyrX (roll rate)', 'GyrY (pitch rate)', 'GyrZ (yaw rate)'],
        'ylabel': 'Vel. Angular [rad/s]',
        'filename': 'IMU_gyro',
    },
    {
        'name': 'Barômetro — Altitude',
        'msg_type': 'BARO',
        'fields': ['Alt'],
        'labels': ['Altitude barométrica'],
        'ylabel': 'Altitude [m]',
        'filename': 'BARO_alt',
    },
    {
        'name': 'Barômetro — Pressão e Temperatura',
        'msg_type': 'BARO',
        'fields': ['Press', 'Temp'],
        'labels': ['Pressão [Pa]', 'Temperatura [°C]'],
        'ylabel': 'Valor',
        'filename': 'BARO_press_temp',
        'twin_axis': True,  # segundo eixo Y para temperatura
    },
    {
        'name': 'GPS — Velocidade',
        'msg_type': 'GPS',
        'fields': ['Spd'],
        'labels': ['Velocidade GPS'],
        'ylabel': 'Velocidade [m/s]',
        'filename': 'GPS_speed',
    },
    {
        'name': 'Vibração',
        'msg_type': 'VIBE',
        'fields': ['VibeX', 'VibeY', 'VibeZ'],
        'labels': ['VibeX', 'VibeY', 'VibeZ'],
        'ylabel': 'Vibração [m/s²]',
        'filename': 'VIBE',
    },
    {
        'name': 'Bateria',
        'msg_type': 'BAT',
        'fields': ['Volt', 'Curr'],
        'labels': ['Tensão [V]', 'Corrente [A]'],
        'ylabel': 'Valor',
        'filename': 'BAT',
        'twin_axis': True,
    },
    {
        'name': 'Potência',
        'msg_type': 'POWR',
        'fields': ['Vcc', 'VServo'],
        'labels': ['Vcc [V]', 'VServo [V]'],
        'ylabel': 'Tensão [V]',
        'filename': 'POWR',
    },
]


def load_bin(filepath):
    """
    Carrega um arquivo .BIN do ArduPilot e retorna um dicionário
    com todos os tipos de mensagem e seus dados.
    """
    from pymavlink import DFReader

    print(f"\nCarregando: {os.path.basename(filepath)}")
    print(f"  Tamanho: {os.path.getsize(filepath)/1024:.1f} KB")

    log = DFReader.DFReader_binary(filepath)
    data = defaultdict(lambda: defaultdict(list))
    msg_counts = defaultdict(int)

    while True:
        m = log.recv_msg()
        if m is None:
            break
        msg_type = m.get_type()
        msg_counts[msg_type] += 1

        # Timestamp (todas as mensagens têm TimeUS)
        if hasattr(m, 'TimeUS'):
            data[msg_type]['TimeUS'].append(m.TimeUS / 1e6)

        # Extrair todos os campos numéricos
        try:
            fields = m.get_fieldnames()
        except:
            continue
        for field in fields:
            if field == 'TimeUS':
                continue
            try:
                val = getattr(m, field)
                if isinstance(val, (int, float)):
                    data[msg_type][field].append(float(val))
            except:
                continue

    # Converter listas para numpy arrays
    for msg_type in data:
        for field in data[msg_type]:
            data[msg_type][field] = np.array(data[msg_type][field])

    # Resumo
    t_start = None
    t_end = None
    for msg_type in data:
        if 'TimeUS' in data[msg_type] and len(data[msg_type]['TimeUS']) > 0:
            t = data[msg_type]['TimeUS']
            if t_start is None or t[0] < t_start:
                t_start = t[0]
            if t_end is None or t[-1] > t_end:
                t_end = t[-1]

    duration = (t_end - t_start) if (t_start and t_end) else 0
    print(f"  Duração: {duration:.1f} s ({duration/60:.1f} min)")
    print(f"  Tipos de mensagem encontrados ({len(msg_counts)}):")
    for mt, count in sorted(msg_counts.items(), key=lambda x: -x[1])[:15]:
        print(f"    {mt:8s}: {count:>6d} amostras")
    if len(msg_counts) > 15:
        print(f"    ... e mais {len(msg_counts)-15} tipos")

    return data, t_start


def print_statistics(data, t_start):
    """Imprime estatísticas dos sensores principais."""
    print("\n" + "=" * 65)
    print("  ESTATÍSTICAS DOS SENSORES")
    print("=" * 65)

    stats_groups = [
        ('ATT', ['Roll', 'Pitch', 'Yaw'], '°'),
        ('IMU', ['AccX', 'AccY', 'AccZ'], 'm/s²'),
        ('IMU', ['GyrX', 'GyrY', 'GyrZ'], 'rad/s'),
        ('BARO', ['Alt'], 'm'),
    ]

    for msg_type, fields, unit in stats_groups:
        if msg_type not in data:
            continue
        available = [f for f in fields if f in data[msg_type]]
        if not available:
            continue
        print(f"\n  {msg_type}:")
        for field in available:
            vals = data[msg_type][field]
            if len(vals) == 0:
                continue
            print(f"    {field:8s}: mean={vals.mean():+10.4f}  "
                  f"std={vals.std():8.4f}  "
                  f"[{vals.min():+.4f}, {vals.max():+.4f}] {unit}")
    print("=" * 65)


def plot_group(data, group, t_start, save=False, out_dir='.', prefix=''):
    """Plota um grupo de dados."""
    msg_type = group['msg_type']

    if msg_type not in data or 'TimeUS' not in data[msg_type]:
        return False

    # Verificar quais campos existem
    available_fields = []
    available_labels = []
    for field, label in zip(group['fields'], group['labels']):
        if field in data[msg_type]:
            available_fields.append(field)
            available_labels.append(label)

    if not available_fields:
        return False

    t = data[msg_type]['TimeUS'] - t_start

    plt.rcParams.update(PLOT_STYLE)
    fig, ax = plt.subplots(figsize=(14, 5))

    if group.get('twin_axis') and len(available_fields) >= 2:
        # Primeiro campo no eixo esquerdo
        ax.plot(t, data[msg_type][available_fields[0]],
                color=COLORS[0], lw=1.0, label=available_labels[0])
        ax.set_ylabel(available_labels[0], color=COLORS[0])
        ax.tick_params(axis='y', labelcolor=COLORS[0])
        ax.legend(loc='upper left', fontsize=9)

        # Segundo campo no eixo direito
        ax2 = ax.twinx()
        ax2.plot(t, data[msg_type][available_fields[1]],
                 color=COLORS[1], lw=1.0, label=available_labels[1])
        ax2.set_ylabel(available_labels[1], color=COLORS[1])
        ax2.tick_params(axis='y', labelcolor=COLORS[1])
        ax2.legend(loc='upper right', fontsize=9)
    else:
        for i, (field, label) in enumerate(zip(available_fields, available_labels)):
            ax.plot(t, data[msg_type][field],
                    color=COLORS[i % len(COLORS)], lw=0.8,
                    label=label, alpha=0.85)
        ax.set_ylabel(group['ylabel'])
        ax.legend(loc='best', fontsize=9, framealpha=0.9)

    ax.set_xlabel('Tempo desde início do log [s]')
    ax.set_title(group['name'], fontsize=12, fontweight='bold')

    plt.tight_layout()

    if save:
        fname = os.path.join(out_dir, f"{prefix}{group['filename']}.pdf")
        plt.savefig(fname, dpi=300, bbox_inches='tight')
        plt.close(fig)
        print(f"  -> {fname}")
    else:
        plt.show()

    return True


def export_csv(data, t_start, out_dir='.', prefix=''):
    """Exporta os dados principais para CSV."""
    import csv

    sensors = ['ATT', 'IMU', 'BARO', 'GPS', 'VIBE', 'BAT']

    for msg_type in sensors:
        if msg_type not in data or 'TimeUS' not in data[msg_type]:
            continue

        fields = sorted([f for f in data[msg_type] if f != 'TimeUS'])
        if not fields:
            continue

        fname = os.path.join(out_dir, f"{prefix}{msg_type}.csv")
        n = len(data[msg_type]['TimeUS'])

        with open(fname, 'w', newline='') as f:
            writer = csv.writer(f)
            writer.writerow(['time_s'] + fields)
            t = data[msg_type]['TimeUS'] - t_start
            for i in range(n):
                row = [f"{t[i]:.6f}"]
                for field in fields:
                    if i < len(data[msg_type][field]):
                        row.append(f"{data[msg_type][field][i]:.6f}")
                    else:
                        row.append('')
                writer.writerow(row)

        print(f"  -> {fname} ({n} linhas)")



def main():
    parser = argparse.ArgumentParser(
        description='Visualizador de logs ArduPilot (.BIN)',
        epilog='Exemplo: python3 plot_bin.py 00000180.BIN --save')
    parser.add_argument('file', nargs='?', default=None,
                        help='Caminho do arquivo .BIN')
    parser.add_argument('--save', action='store_true',
                        help='Salvar PDFs em vez de mostrar interativamente')
    parser.add_argument('--csv', action='store_true',
                        help='Exportar dados para CSV')
    parser.add_argument('--outdir', default='.',
                        help='Pasta de saída (padrão: pasta atual)')
    args = parser.parse_args()

    # Determinar arquivo
    if args.file:
        filepath = args.file
    else:
        # Procurar .BIN na pasta atual
        bins = sorted([f for f in os.listdir('.') if f.endswith('.BIN')])
        if bins:
            print("Arquivos .BIN encontrados:")
            for i, b in enumerate(bins):
                print(f"  [{i}] {b}")
            try:
                idx = int(input("Selecione o número: "))
                filepath = bins[idx]
            except (ValueError, IndexError):
                print("Seleção inválida.")
                return
        else:
            print("Nenhum arquivo .BIN encontrado na pasta atual.")
            print("Uso: python3 plot_bin.py ARQUIVO.BIN")
            return

    if not os.path.isfile(filepath):
        print(f"Arquivo não encontrado: {filepath}")
        return

    # Carregar dados
    data, t_start = load_bin(filepath)

    # Estatísticas
    print_statistics(data, t_start)

    # Prefixo para nomes de arquivo
    prefix = os.path.splitext(os.path.basename(filepath))[0] + '_'

    # Criar pasta de saída
    os.makedirs(args.outdir, exist_ok=True)

    # Exportar CSV
    if args.csv:
        print("\nExportando CSVs...")
        export_csv(data, t_start, args.outdir, prefix)

    # Gerar gráficos
    print(f"\nGerando gráficos {'(salvando PDFs)' if args.save else '(interativo)'}...")
    plotted = 0
    for group in PLOT_GROUPS:
        ok = plot_group(data, group, t_start,
                        save=args.save, out_dir=args.outdir, prefix=prefix)
        if ok:
            plotted += 1

    print(f"\n{plotted} gráficos gerados. Concluído.")


if __name__ == '__main__':
    main()