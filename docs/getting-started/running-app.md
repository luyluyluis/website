import flet as ft
import math
import random
import re
from typing import Dict, List, Optional, Tuple, Any
from __future__ import annotations

class EletrotecnicaSolver:
    def __init__(self):
        self.exercicios = {
            1: "Lei de Ohm (V = R*I)",
            2: "Potência Elétrica (P = V*I)",
            3: "Resistência Equivalente - Série",
            4: "Resistência Equivalente - Paralelo",
            5: "Energia Consumida (E = P*t)",
            6: "Queda de Tensão em Cabos",
            7: "Fator de Potência (FP = P/S)",
            8: "Corrente em Circuito Trifásico",
            9: "Transformação de Tensão (Transformador)",
            10: "Cálculo de Capacitância"
        }
        
        self.unidades = {
            'tensao': 'V',
            'corrente': 'A',
            'resistencia': 'Ω',
            'potencia': 'W',
            'energia': 'kWh',
            'tempo': 'h',
            'comprimento': 'm',
            'area': 'mm²',
            'resistividade': 'Ω.mm²/m',
            'fator_potencia': '',
            'potencia_aparente': 'VA',
            'tensao_fase': 'V',
            'tensao_linha': 'V',
            'relacao_espiras': '',
            'capacitancia': 'F',
            'carga': 'C'
        }
        
        self.quiz = {
            "facil": [
                {
                    "pergunta": "Qual a unidade de medida da resistência elétrica?",
                    "opcoes": ["Volt", "Ohm", "Ampère", "Watt"],
                    "resposta": 1
                },
                {
                    "pergunta": "Qual destes materiais é considerado bom condutor elétrico?",
                    "opcoes": ["Borracha", "Cobre", "Vidro", "Plástico"],
                    "resposta": 1
                }
            ],
            "medio": [
                {
                    "pergunta": "Em um circuito puramente resistivo, a corrente está...",
                    "opcoes": ["Em fase com a tensão", "Atrasada 90° da tensão", "Adiantada 90° da tensão", "Atrasada 180° da tensão"],
                    "resposta": 0
                },
                {
                    "pergunta": "A potência ativa é medida em:",
                    "opcoes": ["Volt-Ampère", "Var", "Watt", "Ohm"],
                    "resposta": 2
                }
            ],
            "dificil": [
                {
                    "pergunta": "Em um circuito série:",
                    "opcoes": ["A corrente é a mesma em todos os componentes", 
                               "A tensão é a mesma em todos os componentes", 
                               "A resistência total é menor que a menor resistência", 
                               "A potência total é zero"],
                    "resposta": 0
                },
                {
                    "pergunta": "Para um circuito trifásico equilibrado, a relação entre tensão de linha e fase é:",
                    "opcoes": ["Vlinha = Vfase", "Vlinha = √3 × Vfase", "Vlinha = Vfase/√3", "Vlinha = 3 × Vfase"],
                    "resposta": 1
                }
            ]
        }
        
        self.jogo_aplicacao = {
            "facil": [
                {
                    "descricao": "Um chuveiro de 5500W em 220V precisa de qual disjuntor?",
                    "parametros": {
                        "fator_correcao": 1.25
                    },
                    "solucao": {
                        "passos": [
                            "Calcular corrente: I = P/V = 5500W / 220V = 25A",
                            "Aplicar fator de correção: 25A * 1.25 = 31.25A",
                            "Arredondar para disjuntor comercial superior: 32A"
                        ],
                        "resposta": "32A"
                    }
                }
            ],
            "medio": [
                {
                    "descricao": "Você precisa alimentar um motor de 5kW a 100m de distância com tensão 220V. Qual seção de cabo usar?",
                    "parametros": {
                        "resistividade": 0.0172,
                        "queda_maxima": 3,
                        "corrente_motor": 23
                    },
                    "solucao": {
                        "passos": [
                            "Calcular queda de tensão máxima permitida: 3% de 220V = 6.6V",
                            "Calcular resistência máxima do cabo: R = V/I = 6.6V / 23A ≈ 0.287Ω",
                            "Calcular seção do cabo: A = (ρ * 2 * L) / R = (0.0172 * 2 * 100) / 0.287 ≈ 12mm²",
                            "Arredondar para seção comercial superior: 16mm²"
                        ],
                        "resposta": "16mm²"
                    }
                }
            ],
            "dificil": [
                {
                    "descricao": "Dimensionar um transformador para alimentar uma carga trifásica de 30kVA com fator de potência 0.8 em 380V",
                    "parametros": {
                        "rendimento": 0.95,
                        "perdas": 500
                    },
                    "solucao": {
                        "passos": [
                            "Calcular potência ativa: P = S * FP = 30kVA * 0.8 = 24kW",
                            "Calcular potência de entrada: Pentrada = (P + Perdas) / Rendimento = (24kW + 0.5kW) / 0.95 ≈ 25.79kW",
                            "Calcular corrente primário para tensão de 13.8kV: I = P / (√3 * V) = 25.79kW / (1.732 * 13.8kV) ≈ 1.08A",
                            "Potência nominal do transformador: 30kVA"
                        ],
                        "resposta": "30kVA"
                    }
                }
            ]
        }
    
    def calcular_lei_ohm(self, dados: Dict[str, float]) -> Dict[str, float]:
        """Calcula valores usando a Lei de Ohm (V = R*I)"""
        resultado = {}
        if 'tensao' in dados and 'resistencia' in dados:
            resultado['corrente'] = dados['tensao'] / dados['resistencia']
        elif 'tensao' in dados and 'corrente' in dados:
            resultado['resistencia'] = dados['tensao'] / dados['corrente']
        elif 'resistencia' in dados and 'corrente' in dados:
            resultado['tensao'] = dados['resistencia'] * dados['corrente']
        return resultado
    
    def calcular_potencia(self, dados: Dict[str, float]) -> Dict[str, float]:
        """Calcula potência elétrica (P = V*I)"""
        resultado = {}
        if 'tensao' in dados and 'corrente' in dados:
            resultado['potencia'] = dados['tensao'] * dados['corrente']
        elif 'tensao' in dados and 'potencia' in dados:
            resultado['corrente'] = dados['potencia'] / dados['tensao']
        elif 'corrente' in dados and 'potencia' in dados:
            resultado['tensao'] = dados['potencia'] / dados['corrente']
        return resultado
    
    def calcular_resistencia_serie(self, resistencias: List[float]) -> float:
        """Calcula resistência equivalente em série"""
        return sum(resistencias)
    
    def calcular_resistencia_paralelo(self, resistencias: List[float]) -> float:
        """Calcula resistência equivalente em paralelo"""
        soma_inversos = sum(1/r for r in resistencias)
        return 1 / soma_inversos
    
    def interpretar_enunciado(self, texto: str) -> Optional[Dict[str, Any]]:
        """Tenta identificar o tipo de exercício a partir do enunciado"""
        texto = texto.lower()
        resultado = {'tipo': None, 'dados': {}}
        
        # Padrões para detecção
        padroes = {
            1: r'tensão|voltagem|volts|v\s*=\s*r\s*\*\s*i|lei\s+de\s+ohm',
            2: r'potência|watts|p\s*=\s*v\s*\*\s*i',
            3: r'série|resistores?\s+em\s+série',
            4: r'paralelo|resistores?\s+em\s+paralelo',
            5: r'energia|consumo|e\s*=\s*p\s*\*\s*t'
        }
        
        # Verifica qual padrão corresponde
        for tipo, padrao in padroes.items():
            if re.search(padrao, texto):
                resultado['tipo'] = tipo
                break
        
        # Extrai valores numéricos do texto
        valores = re.findall(r'(\d+\.?\d*)\s*([a-zA-ZΩ²]+)?', texto)
        for valor, unidade in valores:
            if 'v' in unidade.lower():
                resultado['dados']['tensao'] = float(valor)
            elif 'a' in unidade.lower():
                resultado['dados']['corrente'] = float(valor)
            elif 'Ω' in unidade or 'ohm' in unidade.lower():
                resultado['dados']['resistencia'] = float(valor)
            elif 'w' in unidade.lower():
                resultado['dados']['potencia'] = float(valor)
        
        return resultado if resultado['tipo'] else None

class InterfaceEletrotecnica:
    def __init__(self, page: ft.Page):
        self.page = page
        self.page.title = "Solver de Eletrotécnica"
        self.page.window_width = 900
        self.page.window_height = 700
        self.page.theme_mode = ft.ThemeMode.LIGHT
        self.page.scroll = "adaptive"
        
        self.solver = EletrotecnicaSolver()
        self.num_exercicio = 0
        self.dados_exercicio = {}
        self.controles_dados = {}
        self.quiz_atual = None
        self.jogo_atual = None
        self.pontuacao_quiz = 0
        self.nivel_quiz = "facil"
        self.nivel_jogo = "facil"
        self.erros_quiz = 0
        self.max_erros_quiz = 1
        self.acertos_consecutivos = 0
        
        self.criar_interface()
    
    def criar_interface(self):
        """Cria toda a interface do aplicativo"""
        self.cabecalho = ft.Column(
            controls=[
                ft.Text("Solver de Eletrotécnica", size=24, weight=ft.FontWeight.BOLD),
                ft.Text("Seja bem-vindo ao seu assistente de electrotecnia!", size=16),
                ft.Divider()
            ],
            spacing=10
        )
        
        self.nav_bar = ft.Row(
            controls=[
                ft.ElevatedButton("Resolução de Exercícios", on_click=lambda _: self.mudar_tela("exercicios")),
                ft.ElevatedButton("Interpretar Enunciado", on_click=lambda _: self.mudar_tela("enunciado")),
                ft.ElevatedButton("Quiz de Eletrotécnia", on_click=lambda _: self.mudar_tela("quiz")),
                ft.ElevatedButton("Jogo de Aplicação", on_click=lambda _: self.mudar_tela("jogo")),
                ft.ElevatedButton("Propor Exercícios", on_click=lambda _: self.mudar_tela("propor")),
                ft.ElevatedButton("Sair", on_click=self.sair)
            ],
            alignment=ft.MainAxisAlignment.CENTER,
            spacing=10,
            wrap=True
        )
        
        self.tela_exercicios = self.criar_tela_exercicios()
        self.tela_enunciado = self.criar_tela_enunciado()
        self.tela_quiz = self.criar_tela_quiz()
        self.tela_jogo = self.criar_tela_jogo()
        self.tela_propor = self.criar_tela_propor()
        
        self.btn_voltar = ft.ElevatedButton("Voltar ao Menu", on_click=lambda _: self.mudar_tela("menu"))
        
        self.tela_atual = ft.Column(
            controls=[
                self.cabecalho,
                ft.Text("Selecione uma opção no menu acima:", size=16),
                ft.Image(src="https://via.placeholder.com/600x300?text=Solver+de+Electrotecnia", width=600)
            ],
            spacing=20,
            horizontal_alignment=ft.CrossAxisAlignment.CENTER
        )
        
        self.page.add(
            ft.Column(
                controls=[
                    self.cabecalho,
                    self.nav_bar,
                    self.tela_atual
                ],
                spacing=20,
                expand=True
            )
        )
    
    def criar_tela_exercicios(self) -> ft.Column:
        """Cria a tela de resolução de exercícios"""
        self.dropdown_exercicios = ft.Dropdown(
            options=[ft.dropdown.Option(key=str(k), text=v) 
                    for k, v in self.solver.exercicios.items()],
            label="Selecione o tipo de exercício",
            width=400,
            on_change=self.atualizar_campos_dados
        )
        
        self.container_campos = ft.Column([], spacing=10)
        self.container_solucao = ft.Column([], spacing=10, scroll=ft.ScrollMode.AUTO)
        
        return ft.Column(
            controls=[
                ft.Text("Resolução de Exercícios", size=20, weight=ft.FontWeight.BOLD),
                self.dropdown_exercicios,
                ft.Divider(),
                ft.Text("Informe os dados disponíveis:", size=16),
                self.container_campos,
                ft.Row([
                    ft.ElevatedButton("Resolver Exercício", on_click=self.resolver_exercicio),
                    ft.ElevatedButton("Novo Exercício", on_click=self.limpar_exercicio),
                    self.btn_voltar
                ], spacing=10),
                ft.Divider(),
                ft.Text("Solução:", size=16, weight=ft.FontWeight.BOLD),
                self.container_solucao
            ],
            spacing=15,
            expand=True
        )
    
    def criar_tela_enunciado(self) -> ft.Column:
        """Cria a tela de interpretação de enunciado"""
        self.texto_enunciado = ft.TextField(
            label="Cole o enunciado do exercício aqui",
            multiline=True,
            min_lines=5,
            max_lines=10,
            width=700
        )
        
        self.container_enunciado_solucao = ft.Column([], spacing=10, scroll=ft.ScrollMode.AUTO)
        
        return ft.Column(
            controls=[
                ft.Text("Interpretação de Enunciados", size=20, weight=ft.FontWeight.BOLD),
                self.texto_enunciado,
                ft.Row([
                    ft.ElevatedButton("Interpretar e Resolver", on_click=self.interpretar_enunciado),
                    self.btn_voltar
                ], spacing=10),
                ft.Divider(),
                ft.Text("Resultado:", size=16, weight=ft.FontWeight.BOLD),
                self.container_enunciado_solucao
            ],
            spacing=15,
            expand=True,
            scroll=ft.ScrollMode.AUTO
        )
    
    def criar_tela_quiz(self) -> ft.Column:
        """Cria a tela do quiz de eletrotécnica"""
        self.dropdown_nivel_quiz = ft.Dropdown(
            options=[
                ft.dropdown.Option("facil", "Fácil"),
                ft.dropdown.Option("medio", "Médio"),
                ft.dropdown.Option("dificil", "Difícil")
            ],
            label="Nível de dificuldade",
            value="facil",
            width=200,
            on_change=self.alterar_nivel_quiz
        )
        
        self.pergunta_quiz = ft.Text("", size=18, weight=ft.FontWeight.BOLD)
        self.opcoes_quiz = ft.Column([], spacing=5)
        self.feedback_quiz = ft.Text("", size=16)
        self.pontuacao_texto = ft.Text("Pontuação: 0", size=16)
        self.erros_texto = ft.Text("Erros: 0/1", size=16)
        self.nivel_texto = ft.Text("Nível: Fácil", size=16)
        
        return ft.Column(
            controls=[
                ft.Text("Quiz de Eletrotécnica", size=20, weight=ft.FontWeight.BOLD),
                ft.Row([
                    self.dropdown_nivel_quiz,
                    self.pontuacao_texto,
                    self.erros_texto,
                    self.nivel_texto
                ], spacing=20),
                ft.Text("Teste seus conhecimentos com estas perguntas:", size=16),
                ft.Divider(),
                self.pergunta_quiz,
                self.opcoes_quiz,
                ft.Row([
                    ft.ElevatedButton("Próxima Pergunta", on_click=self.proxima_pergunta_quiz),
                    self.btn_voltar
                ], spacing=10),
                self.feedback_quiz
            ],
            spacing=15,
            expand=True,
            scroll=ft.ScrollMode.AUTO,
            horizontal_alignment=ft.CrossAxisAlignment.CENTER
        )
    
    def criar_tela_jogo(self) -> ft.Column:
        """Cria a tela do jogo de aplicação"""
        self.dropdown_nivel_jogo = ft.Dropdown(
            options=[
                ft.dropdown.Option("facil", "Fácil"),
                ft.dropdown.Option("medio", "Médio"),
                ft.dropdown.Option("dificil", "Difícil")
            ],
            label="Nível de dificuldade",
            value="facil",
            width=200,
            on_change=self.alterar_nivel_jogo
        )
        
        self.titulo_jogo = ft.Text("", size=18, weight=ft.FontWeight.BOLD)
        self.descricao_jogo = ft.Text("", size=16)
        self.parametros_jogo = ft.Column([], spacing=5)
        self.resposta_jogo = ft.TextField(label="Sua resposta", width=300)
        self.feedback_jogo = ft.Column([], spacing=10, scroll=ft.ScrollMode.AUTO)
        self.acertos_texto = ft.Text("Acertos consecutivos: 0", size=16)
        self.nivel_jogo_texto = ft.Text("Nível: Fácil", size=16)
        
        return ft.Column(
            controls=[
                ft.Text("Jogo de Aplicação", size=20, weight=ft.FontWeight.BOLD),
                ft.Row([
                    self.dropdown_nivel_jogo,
                    self.acertos_texto,
                    self.nivel_jogo_texto
                ], spacing=20),
                self.titulo_jogo,
                self.descricao_jogo,
                ft.Divider(),
                self.parametros_jogo,
                self.resposta_jogo,
                ft.Row([
                    ft.ElevatedButton("Verificar Resposta", on_click=self.verificar_resposta_jogo),
                    ft.ElevatedButton("Próximo Desafio", on_click=self.proximo_desafio_jogo),
                    self.btn_voltar
                ], spacing=10),
                ft.Divider(),
                self.feedback_jogo
            ],
            spacing=15,
            expand=True,
            scroll=ft.ScrollMode.AUTO,
            horizontal_alignment=ft.CrossAxisAlignment.CENTER
        )
    
    def criar_tela_propor(self) -> ft.Column:
        """Cria a tela para propor novos exercícios"""
        self.tipo_exercicio = ft.Dropdown(
            options=[ft.dropdown.Option(key=str(k), text=v) 
                    for k, v in self.solver.exercicios.items()],
            label="Tipo de exercício",
            width=400
        )
        
        self.enunciado_proposto = ft.TextField(
            label="Enunciado do exercício",
            multiline=True,
            min_lines=3,
            max_lines=6,
            width=600
        )
        
        self.solucao_proposta = ft.TextField(
            label="Solução esperada",
            multiline=True,
            min_lines=2,
            max_lines=4,
            width=600
        )
        
        self.feedback_propor = ft.Text("", size=16)
        
        return ft.Column(
            controls=[
                ft.Text("Propor Exercícios", size=20, weight=ft.FontWeight.BOLD),
                ft.Text("Contribua com novos exercícios para o banco de dados:", size=16),
                self.tipo_exercicio,
                self.enunciado_proposto,
                self.solucao_proposta,
                ft.Row([
                    ft.ElevatedButton("Enviar Exercício", on_click=self.enviar_exercicio),
                    self.btn_voltar
                ], spacing=10),
                self.feedback_propor
            ],
            spacing=15,
            expand=True,
            scroll=ft.ScrollMode.AUTO,
            horizontal_alignment=ft.CrossAxisAlignment.CENTER
        )
    
    def mudar_tela(self, tela: str) -> None:
        """Muda a tela atual do aplicativo"""
        if tela == "menu":
            self.tela_atual = ft.Column(
                controls=[
                    self.cabecalho,
                    ft.Text("Selecione uma opção no menu acima:", size=16),
                    ft.Image(src="https://via.placeholder.com/600x300?text=Solver+de+Electrotecnia", width=600)
                ],
                spacing=20,
                horizontal_alignment=ft.CrossAxisAlignment.CENTER
            )
        elif tela == "exercicios":
            self.tela_atual = self.tela_exercicios
        elif tela == "enunciado":
            self.tela_atual = self.tela_enunciado
        elif tela == "quiz":
            self.tela_atual = self.tela_quiz
            if not self.quiz_atual:
                self.iniciar_quiz()
        elif tela == "jogo":
            self.tela_atual = self.tela_jogo
            if not self.jogo_atual:
                self.iniciar_jogo()
        elif tela == "propor":
            self.tela_atual = self.tela_propor
        
        self.page.clean()
        self.page.add(
            ft.Column(
                controls=[
                    self.cabecalho,
                    self.nav_bar,
                    self.tela_atual
                ],
                spacing=20,
                expand=True
            )
        )
    
    def alterar_nivel_quiz(self, e: ft.ControlEvent) -> None:
        """Altera o nível de dificuldade do quiz"""
        self.nivel_quiz = self.dropdown_nivel_quiz.value
        self.nivel_texto.value = f"Nível: {self.dropdown_nivel_quiz.value.title()}"
        self.iniciar_quiz()
        self.page.update()
    
    def alterar_nivel_jogo(self, e: ft.ControlEvent) -> None:
        """Altera o nível de dificuldade do jogo"""
        self.nivel_jogo = self.dropdown_nivel_jogo.value
        self.nivel_jogo_texto.value = f"Nível: {self.dropdown_nivel_jogo.value.title()}"
        self.iniciar_jogo()
        self.page.update()
    
    def atualizar_campos_dados(self, e: ft.ControlEvent) -> None:
        """Atualiza os campos de entrada com base no tipo de exercício selecionado"""
        self.num_exercicio = int(self.dropdown_exercicios.value)
        self.container_campos.controls = []
        self.controles_dados = {}
        
        # Configura campos com base no tipo de exercício
        if self.num_exercicio == 1:  # Lei de Ohm
            campos = [
                ("tensao", "Tensão (V)"),
                ("resistencia", "Resistência (Ω)"), 
                ("corrente", "Corrente (A)")
            ]
        elif self.num_exercicio == 2:  # Potência Elétrica
            campos = [
                ("tensao", "Tensão (V)"),
                ("corrente", "Corrente (A)"),
                ("potencia", "Potência (W)")
            ]
        elif self.num_exercicio == 3 or self.num_exercicio == 4:  # Resistência Série/Paralelo
            campos = [
                ("resistencia1", "Resistência 1 (Ω)"),
                ("resistencia2", "Resistência 2 (Ω)"),
                ("resistencia3", "Resistência 3 (Ω) (opcional)"),
                ("resistencia4", "Resistência 4 (Ω) (opcional)")
            ]
        elif self.num_exercicio == 5:  # Energia Consumida
            campos = [
                ("potencia", "Potência (W)"),
                ("tempo", "Tempo (h)"),
                ("energia", "Energia (kWh)")
            ]
        else:
            campos = []
        
        for campo, label in campos:
            self.controles_dados[campo] = ft.TextField(label=label, width=200)
            self.container_campos.controls.append(self.controles_dados[campo])
        
        self.page.update()
    
    def resolver_exercicio(self, e: ft.ControlEvent) -> None:
        """Resolve o exercício com base nos dados fornecidos"""
        self.container_solucao.controls = []
        
        try:
            # Coleta os dados fornecidos
            dados = {}
            for campo, controle in self.controles_dados.items():
                if controle.value:
                    dados[campo] = float(controle.value)
            
            # Executa o cálculo apropriado
            if self.num_exercicio == 1:  # Lei de Ohm
                resultado = self.solver.calcular_lei_ohm({
                    'tensao': dados.get('tensao'),
                    'resistencia': dados.get('resistencia'),
                    'corrente': dados.get('corrente')
                })
                for var, valor in resultado.items():
                    unidade = self.solver.unidades.get(var, '')
                    self.container_solucao.controls.append(
                        ft.Text(f"{var.title()}: {valor:.2f} {unidade}")
                    )
            
            elif self.num_exercicio == 2:  # Potência Elétrica
                resultado = self.solver.calcular_potencia({
                    'tensao': dados.get('tensao'),
                    'corrente': dados.get('corrente'),
                    'potencia': dados.get('potencia')
                })
                for var, valor in resultado.items():
                    unidade = self.solver.unidades.get(var, '')
                    self.container_solucao.controls.append(
                        ft.Text(f"{var.title()}: {valor:.2f} {unidade}")
                    )
            
            elif self.num_exercicio == 3:  # Resistência Série
                resistencias = [dados[f'resistencia{i}'] for i in range(1, 5) if f'resistencia{i}' in dados]
                if len(resistencias) < 2:
                    raise ValueError("Forneça pelo menos 2 resistências")
                
                req = self.solver.calcular_resistencia_serie(resistencias)
                self.container_solucao.controls.append(
                    ft.Text(f"Resistência equivalente em série: {req:.2f} Ω")
                )
            
            elif self.num_exercicio == 4:  # Resistência Paralelo
                resistencias = [dados[f'resistencia{i}'] for i in range(1, 5) if f'resistencia{i}' in dados]
                if len(resistencias) < 2:
                    raise ValueError("Forneça pelo menos 2 resistências")
                
                req = self.solver.calcular_resistencia_paralelo(resistencias)
                self.container_solucao.controls.append(
                    ft.Text(f"Resistência equivalente em paralelo: {req:.2f} Ω")
                )
            
            elif self.num_exercicio == 5:  # Energia Consumida
                if 'potencia' in dados and 'tempo' in dados:
                    energia = (dados['potencia'] * dados['tempo']) / 1000  # Converter para kWh
                    self.container_solucao.controls.append(
                        ft.Text(f"Energia consumida: {energia:.2f} kWh")
                    )
                elif 'energia' in dados and 'tempo' in dados:
                    potencia = (dados['energia'] * 1000) / dados['tempo']
                    self.container_solucao.controls.append(
                        ft.Text(f"Potência: {potencia:.2f} W")
                    )
                elif 'potencia' in dados and 'energia' in dados:
                    tempo = (dados['energia'] * 1000) / dados['potencia']
                    self.container_solucao.controls.append(
                        ft.Text(f"Tempo necessário: {tempo:.2f} horas")
                    )
                else:
                    raise ValueError("Forneça pelo menos dois valores")
            
            else:
                self.container_solucao.controls.append(
                    ft.Text("Exercício ainda não implementado", color=ft.colors.RED)
                )
        
        except Exception as e:
            self.container_solucao.controls.append(
                ft.Text(f"Erro: {str(e)}", color=ft.colors.RED)
            )
        
        self.page.update()
    
    def interpretar_enunciado(self, e: ft.ControlEvent) -> None:
        """Tenta interpretar um enunciado de exercício"""
        self.container_enunciado_solucao.controls = []
        texto = self.texto_enunciado.value
        
        resultado = self.solver.interpretar_enunciado(texto)
        
        if resultado:
            self.container_enunciado_solucao.controls.append(
                ft.Text(f"Tipo de exercício identificado: {self.solver.exercicios[resultado['tipo']]}", 
                       weight=ft.FontWeight.BOLD)
            )
            
            if resultado['dados']:
                self.container_enunciado_solucao.controls.append(
                    ft.Text("Dados identificados:", weight=ft.FontWeight.BOLD)
                )
                for dado, valor in resultado['dados'].items():
                    unidade = self.solver.unidades.get(dado, '')
                    self.container_enunciado_solucao.controls.append(
                        ft.Text(f"- {dado.title()}: {valor} {unidade}")
                    )
            
            # Sugere como resolver
            self.container_enunciado_solucao.controls.append(
                ft.Text("\nComo resolver:", weight=ft.FontWeight.BOLD)
            )
            
            if resultado['tipo'] == 1:
                self.container_enunciado_solucao.controls.append(
                    ft.Text("Use a Lei de Ohm: V = R × I")
                )
                self.container_enunciado_solucao.controls.append(
                    ft.Text("Onde: V = Tensão, R = Resistência, I = Corrente")
                )
            elif resultado['tipo'] == 2:
                self.container_enunciado_solucao.controls.append(
                    ft.Text("Use a fórmula de potência: P = V × I")
                )
        else:
            self.container_enunciado_solucao.controls.append(
                ft.Text("Não foi possível identificar o tipo de exercício", color=ft.colors.RED)
            )
        
        self.page.update()
    
    def limpar_exercicio(self, e: ft.ControlEvent) -> None:
        """Limpa os campos do exercício atual para um novo"""
        self.container_solucao.controls = []
        for controle in self.controles_dados.values():
            controle.value = ""
        self.page.update()
    
    def iniciar_quiz(self) -> None:
        """Inicia o quiz com a primeira pergunta"""
        self.quiz_atual = self.solver.quiz[self.nivel_quiz].copy()
        random.shuffle(self.quiz_atual)
        self.pontuacao_quiz = 0
        self.erros_quiz = 0
        self.pontuacao_texto.value = "Pontuação: 0"
        self.erros_texto.value = f"Erros: {self.erros_quiz}/{self.max_erros_quiz}"
        self.mostrar_pergunta_quiz()
    
    def mostrar_pergunta_quiz(self) -> None:
        """Mostra a próxima pergunta do quiz"""
        if not self.quiz_atual or self.erros_quiz >= self.max_erros_quiz:
            mensagem = "Quiz concluído!" if self.erros_quiz < self.max_erros_quiz else "Você errou demais! Quiz reiniciado."
            self.pergunta_quiz.value = mensagem
            self.opcoes_quiz.controls = []
            self.feedback_quiz.value = f"Pontuação final: {self.pontuacao_quiz}/{len(self.solver.quiz[self.nivel_quiz])}"
            self.iniciar_quiz()  # Reinicia automaticamente
            self.page.update()
            return
        
        pergunta = self.quiz_atual[0]
        self.pergunta_quiz.value = pergunta["pergunta"]
        
        opcoes = []
        for i, opcao in enumerate(pergunta["opcoes"]):
            btn = ft.ElevatedButton(
                opcao,
                data=i,
                on_click=self.verificar_resposta_quiz,
                width=400
            )
            opcoes.append(btn)
        
        self.opcoes_quiz.controls = opcoes
        self.feedback_quiz.value = ""
        self.page.update()
    
    def verificar_resposta_quiz(self, e: ft.ControlEvent) -> None:
        """Verifica se a resposta do quiz está correta"""
        if not self.quiz_atual:
            return
            
        pergunta = self.quiz_atual[0]
        resposta_usuario = int(e.control.data)
        resposta_correta = pergunta["resposta"]
        
        if resposta_usuario == resposta_correta:
            self.feedback_quiz.value = "Correto! ✅"
            self.feedback_quiz.color = ft.colors.GREEN
            self.pontuacao_quiz += 1
            self.pontuacao_texto.value = f"Pontuação: {self.pontuacao_quiz}"
        else:
            self.feedback_quiz.value = f"Incorreto. ❌ A resposta correta é: {pergunta['opcoes'][resposta_correta]}"
            self.feedback_quiz.color = ft.colors.RED
            self.erros_quiz += 1
            self.erros_texto.value = f"Erros: {self.erros_quiz}/{self.max_erros_quiz}"
        
        # Desativa os botões após responder
        for btn in self.opcoes_quiz.controls:
            btn.disabled = True
        
        self.page.update()
    
    def proxima_pergunta_quiz(self, e: ft.ControlEvent) -> None:
        """Vai para a próxima pergunta do quiz"""
        if self.quiz_atual:
            self.quiz_atual.pop(0)
        
        self.mostrar_pergunta_quiz()
    
    def iniciar_jogo(self) -> None:
        """Inicia o jogo de aplicação"""
        cenario = random.choice(self.solver.jogo_aplicacao[self.nivel_jogo])
        self.jogo_atual = cenario
        self
