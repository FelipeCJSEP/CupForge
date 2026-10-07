# ⚖️ Regras de Negócio - CupForge

**Versão:** 1.0  
**Data:** Outubro de 2026  
**Status:** Em Revisão  
**Autor:** Felipe Batista de Assis  

---

## 1. Introdução

Este documento detalha as regras de negócio oficiais que regem a plataforma **CupForge**. Todas as regras aqui descritas devem ser modeladas de forma expressiva dentro da camada de domínio (`CupForge.Domain`), prioritariamente encapsuladas em Entidades Ricas, Agregados e Value Objects, garantindo que o estado do sistema nunca fique inválido.

O sistema é desenhado de forma agnóstica para suportar tanto **Futebol Real** (campo, society, futsal) quanto **Futebol Virtual / eSports** (EA Sports FC, eFootball, etc.), adaptando conceitos como locais (estádios reais vs. plataformas/consoles), participantes (clubes vs. pro-players/clãs) e formatos de série (jogo único, ida e volta, ou séries MD3/MD5).

---

## 2. Regras de Competições e Tabela (RN-COMP)

### RN-COMP-01: Pontuação Parametrizável por Competição
- O sistema deve permitir que o organizador configure os valores de pontuação no regulamento da competição.
- **Valores Padrão Sugeridos:**
  - Vitória: 3 pontos
  - Empate: 1 ponto
  - Derrota: 0 pontos
  - W.O. (Walkover): 3 pontos e placar de 3 a 0 a favor da equipe presente (derrotada com 0 pontos e saldo de -3).
- **Personalização Permitida:** O organizador pode alterar livremente esses valores (ex: 2 pontos por vitória em ligas retrô/amadoras, ponto extra por vitória nos pênaltis, bônus de gols, etc.).

### RN-COMP-02: Critérios de Desempate 100% Flexíveis (Pontos Corridos)
- O usuário possui **total liberdade** para compor, ordenar, habilitar ou desabilitar a lista de critérios de desempate de cada fase ou edição de competição.
- O motor de classificação avaliará os critérios estritamente na **ordem de prioridade estabelecida pelo usuário**.
- **Catálogo de Critérios Disponíveis para Seleção e Ordenação:**
  1. Número de Pontos Ganhos
  2. Número de Vitórias (V)
  3. Saldo de Gols (SG)
  4. Gols Pró / Marcados (GP)
  5. **Confronto Direto (Minitabela / "Mini-campeonato"):**
     - Se o empate ocorrer entre 2 equipes, computam-se os resultados dos jogos disputados exclusivamente entre elas.
     - Se o empate envolver **3 ou mais equipes**, o sistema isola exclusivamente as partidas disputadas entre esse grupo e gera uma **"Minitabela / Mini-campeonato"**, avaliando:
       - Pontos obtidos nas partidas exclusivamente entre as equipes empatadas;
       - Saldo de gols nas partidas exclusivamente entre as equipes empatadas;
       - Gols marcados nas partidas exclusivamente entre as equipes empatadas.
     - Caso um subgrupo permaneça empatado após a minitabela, o processo reavalia recursivamente ou avança para o próximo critério configurado.
  6. Saldo de Gols no Confronto Direto
  7. Gols Marcados Fora de Casa (Geral)
  8. Gols Marcados Fora de Casa (Confronto Direto)
  9. Menor Número de Cartões Vermelhos (Fair Play)
  10. Menor Número de Cartões Amarelos (Fair Play)
  11. Sorteio Oficial
  12. Jogo Extra de Desempate (Playoff de Desempate)
- O usuário pode arrastar/reordenar qualquer um desses itens, definir pesos ou omitir critérios que sua liga não utilize.

### RN-COMP-03: Formatos e Regulamentos 100% Customizáveis pelo Usuário
O usuário tem total autonomia para desenhar a estrutura de qualquer competição, combinando fases de grupos e/ou mata-matas como desejar:

- **Fases de Grupos (Totalmente Customizáveis):**
  - Quantidade de grupos (ex: 2, 4, 8 ou número livre).
  - Quantidade de clubes por grupo.
  - Formato de disputa dentro do grupo:
    - Turno único (todos contra todos dentro do grupo);
    - Turno e returno;
    - Grupos cruzados (clubes do Grupo A jogam exclusivamente contra clubes do Grupo B, etc., como nos campeonatos estaduais).
  - Número de equipes que se classificam por grupo (ex: top 2, top 4, melhores 3º colocados no geral).
  - Regras de rebaixamento ou repescagem a partir da fase de grupos.

- **Fases de Mata-Mata / Eliminatórias (Totalmente Customizáveis):**
  - **Estrutura de Chaveamento:** Chaveamento olímpico fixo predefinido pelo usuário (ex: 1ºA x 2ºB), sorteio livre a cada fase, ou re-seeding (chaveamento reordenado por melhor campanha geral a cada fase).
  - **Formato por Fase:** O usuário pode definir regras diferentes para cada fase (ex: oitavas e quartas em ida e volta; final em jogo único em campo neutro; ou séries melhor de 3).
  - **Mando de Campo:** Definido pelo usuário (sorteio, escolha do time de melhor campanha, ou campo neutro pré-fixado).
  - **Critérios de Desempate da Fase:**
    - Prorrogação (tempo regulável, ex: 2x15 min ou 2x10 min) seguida ou não de pênaltis.
    - Decisão direta por cobranças de pênaltis.
    - Regra do gol qualificado fora de casa (ativável/desativável).
    - Vantagem do empate para o clube de melhor campanha geral ou do mandante (sem pênaltis), se o usuário optar.

### RN-COMP-04: Equilíbrio de Mandos de Campo
- O algoritmo gerador de tabela (Round Robin) busca alternar de forma equilibrada as partidas como mandante e visitante para cada participante ao longo das rodadas da fase.
- Em jogos reais ou partidas virtuais, a indicação do local/servidor é um dado informativo opcional associado à partida.

---

## 3. Regras de Inscrição e Elegibilidade (RN-ELEG)

### RN-ELEG-01: Vínculo Ativo com a Equipe
- Um jogador só pode ser relacionado para uma partida se estiver cadastrado no elenco da equipe e devidamente inscrito na edição da competição.
- O sistema mantém o histórico da relação atleta-clube (data de entrada e saída da equipe).

### RN-ELEG-02: Limites de Atletas Inscritos
- Cada competição/edição pode definir no seu regulamento o número máximo de atletas inscritos por equipe (ex: cota de até 30 atletas).
- O sistema bloqueia a inscrição de atletas adicionais quando a cota estiver esgotada, a menos que o regulamento permita substituição na lista.

### RN-ELEG-03: Atuação por Mais de uma Equipe na Mesma Edição
- Conforme configurado no regulamento da competição:
  - Pode proibir que um atleta que já jogou por um clube atue por outro na mesma edição; ou
  - Pode permitir transferências de lista se o regulamento da liga assim autorizar.

---

## 4. Regras de Partida e Súmula Eletrônica (RN-MATCH)

### RN-MATCH-01: Gestão de Elenco e Escalação Opcional e Flexível
O sistema suporta tanto campeonatos simplificados (apenas confronto e placar entre equipes) quanto campeonatos com gestão detalhada de jogadores:

- **Modo Simplificado (Apenas Equipes e Placar):**
  - O usuário **não é obrigado a cadastrar ou relacionar jogadores**.
  - O torneio pode ser gerido registrando apenas os confrontos e os placares (ex: Equipe A 2 x 1 Equipe B), gerando classificação normalmente.

- **Modo com Escalação / Jogadores (100% Flexível):**
  - **Padrão Sugerido (Futebol de Campo):** 11 titulares e até 12 reservas.
  - **Customização Total pelo Usuário no Regulamento:**
    - **Número de Titulares:** Livremente configurável (ex: 5 para Futsal, 7 para Society, 3 para Futebol 3x3/X1, 11 para Campo tradicional, etc.).
    - **Número Máximo de Reservas:** Configurável (ou sem limite).
    - **Obrigatoriedade de Goleiro:** Configurável (sim/não).
    - **Capitão:** Opcional.
  - Caso haja controle de jogadores na partida, nenhum atleta relacionado pode estar sob suspensão disciplinar ativa.

### RN-MATCH-02: Substituições Parametrizáveis por Regulamento
O sistema disponibiliza as diretrizes modernas da IFAB como configuração padrão, mas concede **total liberdade ao usuário** para definir as regras de substituição de sua competição (adequando-se a futebol profissional, torneios amadores, master ou de base):

- **Configuração Padrão Sugerida (IFAB):**
  - Até **5 substituições** por equipe no tempo normal.
  - Máximo de **3 paradas técnicas** no tempo normal (substituições feitas no intervalo não contam como parada).
  - Em prorrogação: concessão opcional de +1 substituição e +1 parada adicional.
  - Proibição de reentrada do jogador substituído (`PermitirReentrada = false`).

- **Customização Total pelo Usuário no Regulamento:**
  - **Número Máximo de Substituições:** Parametrizável (ex: 3, 5, 7, ilimitadas).
  - **Número Máximo de Paradas (Janelas de Substituição):** Parametrizável (ex: 3 paradas, sem limite de paradas, ou apenas no intervalo).
  - **Substituição Adicional em Prorrogação:** Habilitável / desabilitável e quantidade configurável.
  - **Substituição por Concussão Cerebral:** Opção de habilitar cota extra para concussão sem debitar das paradas normais.
  - **Regra de Reentrada (Substituição Volante):** Possibilidade de habilitar reentrada de atletas que já haviam saído do campo (comum em futebol society, amador e de base).

### RN-MATCH-03: Encerramento de Partida e Apuração de Resultado
- As partidas transitam pelos status: `Agendada` -> `EmAndamento` -> `Encerrada` (podendo assumir `Adiada` ou `Cancelada`).
- Ao marcar a partida como `Encerrada`, o resultado final consolida os pontos na tabela de classificação.
- O organizador pode reabrir ou editar o placar e eventos de uma partida caso haja correções necessárias, recalculando automaticamente as pontuações e estatísticas associadas.

---

## 5. Regras Disciplinares e Suspensões Parametrizáveis (RN-DISC)

O sistema disponibiliza o modelo disciplinar tradicional como padrão sugerido, mas concede **total liberdade ao usuário** para configurar o controle de cartões e suspensões da sua competição:

### RN-DISC-01: Controle de Cartões em Partida
- **Comportamento Padrão Sugerido:**
  - 2 cartões amarelos para o mesmo atleta na mesma partida = Cartão Vermelho Indireto (expulsão).
- **Personalização pelo Usuário:**
  - Habilitar/desabilitar uso de cartões disciplinares na competição.
  - Suporte a cartão azul/temporário (comum em futebol society/amador, ex: suspensão de 2 ou 5 minutos).

### RN-DISC-02: Critérios de Suspensão Automática por Cartões
- **Modelo Padrão Sugerido:**
  - **Cartão Vermelho:** Suspensão automática de 1 partida.
  - **Acúmulo de Cartões Amarelos:** Suspensão a cada **3 cartões amarelos** acumulados.
- **Customização Total pelo Usuário no Regulamento:**
  - **Limite de Amarelos para Suspensão:** Configurável (ex: 2, 3, 5 amarelos, ou desativar suspensão por acúmulo de amarelos).
  - **Quantidade de Jogos de Suspensão:** Configurável por tipo de cartão (ex: 1 jogo para vermelho indireto, 2 jogos para vermelho direto).
  - **Independência de Cartões:** Configurar se o cartão vermelho anula ou não os amarelos acumulados daquela partida.

### RN-DISC-03: Zera / Anistia de Cartões em Fases Específicas
- **Modelo Padrão Sugerido:**
  - Zera de cartões amarelos acumulados ao fim da fase de grupos para quem não atingiu a cota de suspensão.
- **Customização pelo Usuário:**
  - O usuário escolhe livremente se haverá ou não zera de cartões, e em qual fase ela ocorrerá (ex: após fase de grupos, após quartas de final, ou nunca zerar).
  - Opção de anistiar ou não suspensões pendentes para a grande final.
