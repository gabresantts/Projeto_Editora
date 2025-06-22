# Projeto_Editora
Projeto acerca de uma editora de livros para a Disciplina: Banco de Dados - DQL e DTL. 

Projeto em Trio

# Discentes: 

Ana Carolina da Silva Santos;
Gabriel Gleydson Lima dos Santos;
Muriel Bezerra da Silva.

Recife, 27 de Junho de 2025.

# Minimundo - Editora

O minimundo da editora modelado no MER e MR descreve o funcionamento de uma empresa do ramo editorial que gerencia de forma integrada o cadastro, publicação, comercialização e controle de exemplares de livros. A editora trabalha com diversos livros, cada um identificado por um ISBN único e contendo informações como título, autor, editora, data de publicação, edição, número de páginas, gênero e descrição. Os autores responsáveis pelas obras possuem dados registrados como nome, biografia, nacionalidade e datas de nascimento e falecimento. Cada livro pode ter vários exemplares, os quais são identificados por um número de série e armazenam informações sobre o estado físico (como disponível, danificado, reservado), localização física, data e custo de aquisição. 

Os livros também são classificados por palavras-chave e por áreas de conhecimento, permitindo múltiplas associações para fins de catalogação e busca. As palavras-chave e áreas são cadastradas com códigos e descrições. A editora mantém um quadro de funcionários, que são vinculados a departamentos internos. Os funcionários possuem informações pessoais e profissionais como nome, cargo, salário, CPF, telefone, e-mail e data de contratação, enquanto os departamentos contêm nome, responsável, orçamento e descrição das atividades desempenhadas. O sistema também contempla o processo de pedidos e vendas, onde clientes realizam pedidos de livros. 

Cada pedido está vinculado a um cliente e pode conter múltiplos itens, representando os exemplares adquiridos, com informações sobre quantidade, preço unitário, desconto e número de série. Os pedidos registram a data da transação, data de entrega, status, forma de pagamento e valor total. Além disso, o sistema permite o acompanhamento do histórico de preços dos livros, registrando alterações com data, valor antigo e novo, motivo da alteração e funcionário responsável pela mudança. Esse conjunto de informações visa fornecer um controle completo sobre a operação da editora, desde o cadastro e catalogação de obras até a gestão de vendas, exemplares físicos e recursos humanos internos.

Para efeito de visualização e comprovação, elaboramos o Diagrama MER pelo BRModelo Web e o Diagrama MR pelo MySQL Workbench com os Códigos e os Scripts solicitados de forma organizada também pelo MySQL. Vejamos a seguir:

Projeto_Editora/
├── imagens/
│   ├── modelagem_conceitual.png
│   └── modelo_logico.png
├── README.md

# 1. Anexar imagem do modelo lógico (Diagrama MER):

Abaixo está o diagrama da modelagem conceitual:

![Modelagem Conceitual](imagens/modelagem_conceitual.png)

# 2. Anexar imagem do modelo físico (Diagrama MR):

Abaixo está o diagrama do modelo lógico:

![Modelo Lógico](imagens/modelo_logico.png)

# 3. Anexar a esse documento os Códigos abaixo de uma forma organizada e bem documentada:

-- Criação do banco de dados 
DROP DATABASE IF EXISTS editora_livros;
CREATE DATABASE editora_livros;
USE editora_livros;

-- TABELAS PRINCIPAIS

CREATE TABLE EDITORA (
    id_editora INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    cnpj VARCHAR(14) UNIQUE,
    endereco VARCHAR(150),
    telefone VARCHAR(20),
    email VARCHAR(100)
);

CREATE TABLE AUTOR (
    id_autor INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    biografia TEXT,
    nacionalidade VARCHAR(50),
    data_nascimento DATE,
    data_falecimento DATE NULL
);

CREATE TABLE DEPARTAMENTO (
    id_departamento INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(50) NOT NULL,
    responsavel VARCHAR(100),
    descricao TEXT,
    orcamento DECIMAL(12,2)
);

-- TABELAS SECUNDÁRIAS E RELACIONAMENTOS
CREATE TABLE LIVRO (
    isbn VARCHAR(13) PRIMARY KEY,
    titulo VARCHAR(100) NOT NULL,
    id_autor INT NOT NULL,
    id_editora INT NOT NULL,
    data_publicacao DATE,
    edicao INT DEFAULT 1,
    num_paginas INT CHECK (num_paginas > 0),
    preco_sugerido DECIMAL(10,2),
    descricao TEXT,
    FOREIGN KEY (id_autor) REFERENCES AUTOR(id_autor),
    FOREIGN KEY (id_editora) REFERENCES EDITORA(id_editora)
);

CREATE TABLE FUNCIONARIO (
    id_funcionario INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    cpf VARCHAR(11) UNIQUE,
    cargo VARCHAR(50) NOT NULL,
    salario DECIMAL(10,2),
    telefone VARCHAR(20),
    email VARCHAR(100),
    endereco VARCHAR(150),
    id_departamento INT NOT NULL,
    data_contratacao DATE DEFAULT (CURRENT_DATE),
    FOREIGN KEY (id_departamento) REFERENCES DEPARTAMENTO(id_departamento)
);

CREATE TABLE AREA_CONHECIMENTO (
    id_area INT PRIMARY KEY AUTO_INCREMENT,
    codigo VARCHAR(10) UNIQUE,
    descricao VARCHAR(100) NOT NULL
);

CREATE TABLE PALAVRA_CHAVE (
    id_palavra INT PRIMARY KEY AUTO_INCREMENT,
    codigo VARCHAR(10) UNIQUE,
    descricao VARCHAR(50) NOT NULL
);

-- TABELAS DE RELACIONAMENTOS MUITOS-PARA-MUITOS


CREATE TABLE LIVRO_AREA (
    isbn VARCHAR(13),
    id_area INT,
    PRIMARY KEY (isbn, id_area),
    FOREIGN KEY (isbn) REFERENCES LIVRO(isbn) ON DELETE CASCADE,
    FOREIGN KEY (id_area) REFERENCES AREA_CONHECIMENTO(id_area) ON DELETE CASCADE
);

CREATE TABLE LIVRO_PALAVRA (
    isbn VARCHAR(13),
    id_palavra INT,
    PRIMARY KEY (isbn, id_palavra),
    FOREIGN KEY (isbn) REFERENCES LIVRO(isbn) ON DELETE CASCADE,
    FOREIGN KEY (id_palavra) REFERENCES PALAVRA_CHAVE(id_palavra) ON DELETE CASCADE
);

-- TABELAS DE ESTOQUE E VENDAS

CREATE TABLE EXEMPLAR (
    numero_serie INT PRIMARY KEY AUTO_INCREMENT,
    isbn VARCHAR(13) NOT NULL,
    estado ENUM('disponivel', 'reservado', 'emprestado', 'danificado', 'perdido') DEFAULT 'disponivel',
    localizacao VARCHAR(100),
    data_aquisicao DATE DEFAULT (CURRENT_DATE),
    custo_aquisicao DECIMAL(10,2),
    FOREIGN KEY (isbn) REFERENCES LIVRO(isbn),
    CHECK (custo_aquisicao >= 0)
);

CREATE TABLE CLIENTE (
    id_cliente INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    tipo ENUM('pessoa_fisica', 'pessoa_juridica') NOT NULL,
    cpf_cnpj VARCHAR(14) UNIQUE,
    email VARCHAR(100),
    telefone VARCHAR(20),
    endereco VARCHAR(150),
    data_cadastro DATE DEFAULT (CURRENT_DATE)
);

CREATE TABLE PEDIDO (
    id_pedido INT PRIMARY KEY AUTO_INCREMENT,
    id_cliente INT NOT NULL,
    id_funcionario INT,
    data_pedido DATETIME DEFAULT CURRENT_TIMESTAMP,
    data_entrega DATE,
    status ENUM('pendente', 'processando', 'enviado', 'entregue', 'cancelado') DEFAULT 'pendente',
    forma_pagamento ENUM('credito', 'debito', 'boleto', 'transferencia', 'pix'),
    valor_total DECIMAL(12,2),
    FOREIGN KEY (id_cliente) REFERENCES CLIENTE(id_cliente),
    FOREIGN KEY (id_funcionario) REFERENCES FUNCIONARIO(id_funcionario)
);

CREATE TABLE ITEM_PEDIDO (
    id_pedido INT,
    numero_serie INT,
    preco_unitario DECIMAL(10,2) NOT NULL,
    quantidade INT DEFAULT 1,
    desconto DECIMAL(5,2) DEFAULT 0,
    PRIMARY KEY (id_pedido, numero_serie),
    FOREIGN KEY (id_pedido) REFERENCES PEDIDO(id_pedido) ON DELETE CASCADE,
    FOREIGN KEY (numero_serie) REFERENCES EXEMPLAR(numero_serie),
    CHECK (preco_unitario >= 0),
    CHECK (quantidade > 0),
    CHECK (desconto BETWEEN 0 AND 100)
);

-- TABELA DE HISTÓRICO (AUDITORIA)

CREATE TABLE HISTORICO_PRECO (
    id_historico INT PRIMARY KEY AUTO_INCREMENT,
    isbn VARCHAR(13) NOT NULL,
    preco_antigo DECIMAL(10,2) NOT NULL,
    preco_novo DECIMAL(10,2) NOT NULL,
    data_alteracao DATETIME DEFAULT CURRENT_TIMESTAMP,
    id_funcionario INT,
    motivo VARCHAR(255),
    FOREIGN KEY (isbn) REFERENCES LIVRO(isbn),
    FOREIGN KEY (id_funcionario) REFERENCES FUNCIONARIO(id_funcionario)
);

# 3.1 Anexar a esse documento os Scripts abaixo de uma forma organizada e bem documentada:

# 3.1.1 Script de Criação (DDL) de Tabelas e Views

-- DROP e CRIAÇÃO DO BANCO
DROP DATABASE IF EXISTS editora_livros;
CREATE DATABASE editora_livros;
USE editora_livros;

-- TABELAS PRINCIPAIS
CREATE TABLE EDITORA (
    id_editora INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    cnpj VARCHAR(14) UNIQUE,
    endereco VARCHAR(150),
    telefone VARCHAR(20),
    email VARCHAR(100)
);

CREATE TABLE AUTOR (
    id_autor INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    biografia TEXT,
    nacionalidade VARCHAR(50),
    data_nascimento DATE,
    data_falecimento DATE NULL
);

CREATE TABLE DEPARTAMENTO (
    id_departamento INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(50) NOT NULL,
    responsavel VARCHAR(100),
    descricao TEXT,
    orcamento DECIMAL(12,2)
);

-- TABELAS SECUNDÁRIAS
CREATE TABLE LIVRO (
    isbn VARCHAR(13) PRIMARY KEY,
    titulo VARCHAR(100) NOT NULL,
    id_autor INT NOT NULL,
    id_editora INT NOT NULL,
    data_publicacao DATE,
    edicao INT DEFAULT 1,
    num_paginas INT CHECK (num_paginas > 0),
    preco_sugerido DECIMAL(10,2),
    descricao TEXT,
    FOREIGN KEY (id_autor) REFERENCES AUTOR(id_autor),
    FOREIGN KEY (id_editora) REFERENCES EDITORA(id_editora)
);

CREATE TABLE FUNCIONARIO (
    id_funcionario INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    cpf VARCHAR(11) UNIQUE,
    cargo VARCHAR(50) NOT NULL,
    salario DECIMAL(10,2),
    telefone VARCHAR(20),
    email VARCHAR(100),
    endereco VARCHAR(150),
    id_departamento INT NOT NULL,
    data_contratacao DATE DEFAULT (CURRENT_DATE),
    FOREIGN KEY (id_departamento) REFERENCES DEPARTAMENTO(id_departamento)
);

CREATE TABLE AREA_CONHECIMENTO (
    id_area INT PRIMARY KEY AUTO_INCREMENT,
    codigo VARCHAR(10) UNIQUE,
    descricao VARCHAR(100) NOT NULL
);

CREATE TABLE PALAVRA_CHAVE (
    id_palavra INT PRIMARY KEY AUTO_INCREMENT,
    codigo VARCHAR(10) UNIQUE,
    descricao VARCHAR(50) NOT NULL
);

-- RELACIONAMENTOS MUITOS-PARA-MUITOS
CREATE TABLE LIVRO_AREA (
    isbn VARCHAR(13),
    id_area INT,
    PRIMARY KEY (isbn, id_area),
    FOREIGN KEY (isbn) REFERENCES LIVRO(isbn) ON DELETE CASCADE,
    FOREIGN KEY (id_area) REFERENCES AREA_CONHECIMENTO(id_area) ON DELETE CASCADE
);

CREATE TABLE LIVRO_PALAVRA (
    isbn VARCHAR(13),
    id_palavra INT,
    PRIMARY KEY (isbn, id_palavra),
    FOREIGN KEY (isbn) REFERENCES LIVRO(isbn) ON DELETE CASCADE,
    FOREIGN KEY (id_palavra) REFERENCES PALAVRA_CHAVE(id_palavra) ON DELETE CASCADE
);

-- ESTOQUE E VENDAS
CREATE TABLE EXEMPLAR (
    numero_serie INT PRIMARY KEY AUTO_INCREMENT,
    isbn VARCHAR(13) NOT NULL,
    estado ENUM('disponivel', 'reservado', 'emprestado', 'danificado', 'perdido') DEFAULT 'disponivel',
    localizacao VARCHAR(100),
    data_aquisicao DATE DEFAULT (CURRENT_DATE),
    custo_aquisicao DECIMAL(10,2),
    FOREIGN KEY (isbn) REFERENCES LIVRO(isbn),
    CHECK (custo_aquisicao >= 0)
);

CREATE TABLE CLIENTE (
    id_cliente INT PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(100) NOT NULL,
    tipo ENUM('pessoa_fisica', 'pessoa_juridica') NOT NULL,
    cpf_cnpj VARCHAR(14) UNIQUE,
    email VARCHAR(100),
    telefone VARCHAR(20),
    endereco VARCHAR(150),
    data_cadastro DATE DEFAULT (CURRENT_DATE)
);

CREATE TABLE PEDIDO (
    id_pedido INT PRIMARY KEY AUTO_INCREMENT,
    id_cliente INT NOT NULL,
    id_funcionario INT,
    data_pedido DATETIME DEFAULT CURRENT_TIMESTAMP,
    data_entrega DATE,
    status ENUM('pendente', 'processando', 'enviado', 'entregue', 'cancelado') DEFAULT 'pendente',
    forma_pagamento ENUM('credito', 'debito', 'boleto', 'transferencia', 'pix'),
    valor_total DECIMAL(12,2),
    FOREIGN KEY (id_cliente) REFERENCES CLIENTE(id_cliente),
    FOREIGN KEY (id_funcionario) REFERENCES FUNCIONARIO(id_funcionario)
);

CREATE TABLE ITEM_PEDIDO (
    id_pedido INT,
    numero_serie INT,
    preco_unitario DECIMAL(10,2) NOT NULL,
    quantidade INT DEFAULT 1,
    desconto DECIMAL(5,2) DEFAULT 0,
    PRIMARY KEY (id_pedido, numero_serie),
    FOREIGN KEY (id_pedido) REFERENCES PEDIDO(id_pedido) ON DELETE CASCADE,
    FOREIGN KEY (numero_serie) REFERENCES EXEMPLAR(numero_serie),
    CHECK (preco_unitario >= 0),
    CHECK (quantidade > 0),
    CHECK (desconto BETWEEN 0 AND 100)
);

-- AUDITORIA
CREATE TABLE HISTORICO_PRECO (
    id_historico INT PRIMARY KEY AUTO_INCREMENT,
    isbn VARCHAR(13) NOT NULL,
    preco_antigo DECIMAL(10,2) NOT NULL,
    preco_novo DECIMAL(10,2) NOT NULL,
    data_alteracao DATETIME DEFAULT CURRENT_TIMESTAMP,
    id_funcionario INT,
    motivo VARCHAR(255),
    FOREIGN KEY (isbn) REFERENCES LIVRO(isbn),
    FOREIGN KEY (id_funcionario) REFERENCES FUNCIONARIO(id_funcionario)
);

# 3.1.2 Scripts para Alterar Estruturas das Tabelas (ALTER TABLE)

-- 1. Adicionar coluna de site à tabela EDITORA
ALTER TABLE EDITORA
ADD COLUMN site VARCHAR(100);

-- 2. Adicionar índice no nome do AUTOR
ALTER TABLE AUTOR
ADD INDEX idx_nome_autor (nome);

-- 3. Aumentar o tamanho da coluna 'codigo' em AREA_CONHECIMENTO
ALTER TABLE AREA_CONHECIMENTO
MODIFY COLUMN codigo VARCHAR(20);

-- 4. Tornar a coluna 'descricao' da tabela DEPARTAMENTO obrigatória
ALTER TABLE DEPARTAMENTO
MODIFY COLUMN descricao TEXT NOT NULL;

-- 5. Adicionar coluna de "estoque_minimo" na tabela LIVRO
ALTER TABLE LIVRO
ADD COLUMN estoque_minimo INT DEFAULT 5;

-- 6. Adicionar coluna "ativo" à tabela FUNCIONARIO
ALTER TABLE FUNCIONARIO
ADD COLUMN ativo BOOLEAN DEFAULT TRUE;

-- 7. Adicionar uma nova coluna "frete" à tabela PEDIDO
ALTER TABLE PEDIDO
ADD COLUMN frete DECIMAL(10,2) DEFAULT 0;

-- 8. Alterar o tipo da coluna 'descricao' em PALAVRA_CHAVE para TEXT
ALTER TABLE PALAVRA_CHAVE
MODIFY COLUMN descricao TEXT NOT NULL;

-- 9. Adicionar coluna "data_alteracao" à tabela CLIENTE para controle de atualização de dados
ALTER TABLE CLIENTE
ADD COLUMN data_alteracao DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP;

-- 10. Adicionar chave estrangeira da editora ao departamento (hipotética relação organizacional)
ALTER TABLE DEPARTAMENTO
ADD COLUMN id_editora INT,
ADD FOREIGN KEY (id_editora) REFERENCES EDITORA(id_editora);

# 3.1.3 Script para Destruir Todos os Objetos do Banco de Dados

USE editora_livros;

-- 1. Remover todas as VIEWS
DROP VIEW IF EXISTS 
    vw_relatorio_vendas_por_livro,
    vw_livros_por_autor,
    vw_clientes_por_tipo,
    vw_pedidos_em_aberto,
    vw_funcionarios_departamento,
    vw_livros_com_estoque,
    vw_historico_precos,
    vw_resumo_clientes,
    vw_livros_por_area,
    vw_estoque_detalhado;

-- 2. Remover todas as tabelas respeitando ordem de dependência
DROP TABLE IF EXISTS 
    ITEM_PEDIDO,
    PEDIDO,
    EXEMPLAR,
    HISTORICO_PRECO,
    LIVRO_PALAVRA,
    LIVRO_AREA,
    PALAVRA_CHAVE,
    AREA_CONHECIMENTO,
    LIVRO,
    FUNCIONARIO,
    CLIENTE,
    AUTOR,
    DEPARTAMENTO,
    EDITORA;

-- 3. (Opcional) Remover stored procedures, functions e triggers
-- Excluir procedures
-- DELIMITER //
-- DROP PROCEDURE IF EXISTS nome_da_procedure;
-- DROP FUNCTION IF EXISTS nome_da_funcao;
-- DROP TRIGGER IF EXISTS nome_do_trigger;
-- DELIMITER ;

-- 4. (Opcional) Dropar o banco inteiro
-- DROP DATABASE IF EXISTS editora_livros;

# 3.1.4 Script de Inserção de Dados (INSERT INTO)

-- 1. Tabela: Editora

INSERT INTO EDITORA (nome, cnpj, endereco, telefone, email, site) VALUES
('Editora Alfa', '12345678000101', 'Rua A, 100', '11999990001', 'contato@alfa.com.br', 'www.alfa.com.br'),
('Editora Beta', '12345678000102', 'Rua B, 200', '11999990002', 'contato@beta.com.br', 'www.beta.com.br'),
('Editora Gama', '12345678000103', 'Rua C, 300', '11999990003', 'contato@gama.com.br', 'www.gama.com.br'),
('Editora Delta', '12345678000104', 'Rua D, 400', '11999990004', 'contato@delta.com.br', 'www.delta.com.br'),
('Editora Épsilon', '12345678000105', 'Rua E, 500', '11999990005', 'contato@epsilon.com.br', 'www.epsilon.com.br'),
('Editora Zeta', '12345678000106', 'Rua F, 600', '11999990006', 'contato@zeta.com.br', 'www.zeta.com.br'),
('Editora Eta', '12345678000107', 'Rua G, 700', '11999990007', 'contato@eta.com.br', 'www.eta.com.br'),
('Editora Theta', '12345678000108', 'Rua H, 800', '11999990008', 'contato@theta.com.br', 'www.theta.com.br'),
('Editora Iota', '12345678000109', 'Rua I, 900', '11999990009', 'contato@iota.com.br', 'www.iota.com.br'),
('Editora Kappa', '12345678000110', 'Rua J, 1000', '11999990010', 'contato@kappa.com.br', 'www.kappa.com.br');

-- 2. Tabela: Autor

INSERT INTO AUTOR (nome, biografia, nacionalidade, data_nascimento, data_falecimento) VALUES
('Ana Silva', 'Autora de ficção científica.', 'Brasileira', '1970-05-12', NULL),
('Carlos Mendes', 'Especialista em história.', 'Português', '1965-02-20', NULL),
('Beatriz Lima', 'Romancista premiada.', 'Brasileira', '1980-11-03', NULL),
('Eduardo Torres', 'Autor técnico.', 'Chileno', '1978-03-25', NULL),
('Fernanda Rocha', 'Autora infantil.', 'Brasileira', '1985-07-15', NULL),
('Gustavo Nogueira', 'Crítico literário.', 'Argentino', '1950-01-10', '2020-10-10'),
('Helena Souza', 'Poetisa contemporânea.', 'Brasileira', '1990-06-30', NULL),
('Igor Pereira', 'Especialista em TI.', 'Brasileiro', '1975-09-05', NULL),
('Joana Teixeira', 'Professora e autora.', 'Moçambicana', '1968-04-22', NULL),
('Lucas Carvalho', 'Jornalista investigativo.', 'Brasileiro', '1982-12-19', NULL);

-- 3. Tabela: Departamento

INSERT INTO DEPARTAMENTO (nome, responsavel, descricao, orcamento, id_editora) VALUES
('Marketing', 'Fernanda Duarte', 'Departamento de marketing e comunicação.', 50000, 1),
('Financeiro', 'Rafael Costa', 'Departamento financeiro.', 75000, 2),
('Editorial', 'Juliana Alves', 'Responsável pela revisão e edição.', 60000, 1),
('TI', 'Marcelo Souza', 'Tecnologia da Informação.', 80000, 3),
('RH', 'Patrícia Lima', 'Recursos Humanos.', 40000, 2),
('Jurídico', 'Carlos Dias', 'Suporte legal e contratos.', 30000, 3),
('Distribuição', 'Lúcio Rocha', 'Distribuição de livros.', 45000, 4),
('Vendas', 'Márcia Andrade', 'Equipe de vendas e suporte.', 55000, 4),
('Pesquisa', 'Helena Braga', 'Pesquisa de mercado.', 32000, 5),
('Logística', 'Tiago Reis', 'Gerenciamento logístico.', 48000, 5);

-- 4. Tabela: Livro

INSERT INTO LIVRO (isbn, titulo, id_autor, id_editora, data_publicacao, edicao, num_paginas, preco_sugerido, descricao, estoque_minimo) VALUES
('9780000000001', 'O Futuro do Amanhã', 1, 1, '2021-05-10', 1, 320, 59.90, 'Uma obra de ficção científica sobre o futuro.', 5),
('9780000000002', 'História Antiga', 2, 2, '2018-03-15', 2, 450, 89.50, 'Um estudo aprofundado sobre civilizações antigas.', 5),
('9780000000003', 'Romance em Paris', 3, 3, '2020-09-01', 1, 280, 42.00, 'Um romance envolvente na capital francesa.', 5),
('9780000000004', 'Redes de Computadores', 4, 4, '2019-11-20', 3, 500, 99.90, 'Livro técnico sobre redes e comunicação.', 5),
('9780000000005', 'Aventuras no Espaço', 5, 5, '2022-01-05', 1, 210, 35.00, 'Livro infantil com aventuras espaciais.', 5),
('9780000000006', 'Crítica Literária Moderna', 6, 6, '2017-06-10', 2, 370, 64.90, 'Análise crítica da literatura atual.', 5),
('9780000000007', 'Poesia Urbana', 7, 7, '2021-08-08', 1, 190, 29.90, 'Poemas inspirados na vida nas cidades.', 5),
('9780000000008', 'Sistemas Operacionais', 8, 1, '2020-10-12', 4, 520, 110.00, 'Conceitos modernos de SO.', 5),
('9780000000009', 'Educação Inclusiva', 9, 2, '2019-02-25', 1, 300, 49.90, 'Práticas pedagógicas para inclusão.', 5),
('9780000000010', 'Corrupção e Política', 10, 3, '2021-04-14', 1, 400, 79.90, 'Investigação jornalística profunda.', 5);

-- 5. Tabela: Funcionario

INSERT INTO FUNCIONARIO (nome, cpf, cargo, salario, telefone, email, endereco, id_departamento, ativo) VALUES
('Mariana Oliveira', '11111111111', 'Analista de Marketing', 4500.00, '11991110001', 'mariana@editora.com', 'Rua 1, nº 10', 1, TRUE),
('Renato Souza', '11111111112', 'Contador', 5500.00, '11991110002', 'renato@editora.com', 'Rua 2, nº 20', 2, TRUE),
('Juliana Mello', '11111111113', 'Editor', 6000.00, '11991110003', 'juliana@editora.com', 'Rua 3, nº 30', 3, TRUE),
('Rodrigo Martins', '11111111114', 'Técnico de TI', 5000.00, '11991110004', 'rodrigo@editora.com', 'Rua 4, nº 40', 4, TRUE),
('Patrícia Gomes', '11111111115', 'Assistente de RH', 4000.00, '11991110005', 'patricia@editora.com', 'Rua 5, nº 50', 5, TRUE),
('Cláudio Nunes', '11111111116', 'Advogado', 7000.00, '11991110006', 'claudio@editora.com', 'Rua 6, nº 60', 6, TRUE),
('Lívia Andrade', '11111111117', 'Supervisor de Distribuição', 4800.00, '11991110007', 'livia@editora.com', 'Rua 7, nº 70', 7, TRUE),
('Márcio Ferreira', '11111111118', 'Vendedor', 3000.00, '11991110008', 'marcio@editora.com', 'Rua 8, nº 80', 8, TRUE),
('Eduarda Silva', '11111111119', 'Analista de Pesquisa', 4700.00, '11991110009', 'eduarda@editora.com', 'Rua 9, nº 90', 9, TRUE),
('Tiago Rocha', '11111111110', 'Motorista', 3500.00, '11991110010', 'tiago@editora.com', 'Rua 10, nº 100', 10, TRUE);

-- 6. Tabela: Cliente

INSERT INTO CLIENTE (nome, tipo, cpf_cnpj, email, telefone, endereco) VALUES
('João Silva', 'pessoa_fisica', '12345678901', 'joao@email.com', '11998880001', 'Av. A, 123'),
('Maria Santos', 'pessoa_fisica', '12345678902', 'maria@email.com', '11998880002', 'Av. B, 456'),
('Empresa X', 'pessoa_juridica', '12345678000191', 'contato@empresax.com', '11998880003', 'Rua Empresarial, 789'),
('Carlos Souza', 'pessoa_fisica', '12345678903', 'carlos@email.com', '11998880004', 'Rua C, 147'),
('Ana Beatriz', 'pessoa_fisica', '12345678904', 'ana@email.com', '11998880005', 'Rua D, 258'),
('Loja Y', 'pessoa_juridica', '12345678000192', 'contato@lojay.com', '11998880006', 'Rua Comercial, 369'),
('Fernanda Lima', 'pessoa_fisica', '12345678905', 'fernanda@email.com', '11998880007', 'Rua E, 321'),
('Gabriel Costa', 'pessoa_fisica', '12345678906', 'gabriel@email.com', '11998880008', 'Rua F, 654'),
('Startup Z', 'pessoa_juridica', '12345678000193', 'hello@startupz.com', '11998880009', 'Av. Tech, 987'),
('Renata Oliveira', 'pessoa_fisica', '12345678907', 'renata@email.com', '11998880010', 'Rua G, 159');

-- 7. Tabela: Exemplar

INSERT INTO EXEMPLAR (isbn, estado, localizacao, custo_aquisicao) VALUES
('9780000000001', 'disponivel', 'Estante A1', 30.00),
('9780000000001', 'emprestado', 'Estante A1', 30.00),
('9780000000002', 'disponivel', 'Estante B1', 50.00),
('9780000000003', 'danificado', 'Estante C1', 25.00),
('9780000000004', 'disponivel', 'Estante D1', 60.00),
('9780000000005', 'reservado', 'Estante E1', 20.00),
('9780000000006', 'disponivel', 'Estante F1', 35.00),
('9780000000007', 'perdido', 'Estante G1', 15.00),
('9780000000008', 'disponivel', 'Estante H1', 70.00),
('9780000000009', 'disponivel', 'Estante I1', 45.00);

-- 8. Tabela: Pedido

INSERT INTO PEDIDO (id_cliente, id_funcionario, data_entrega, status, forma_pagamento, valor_total) VALUES
(1, 1, '2025-05-01', 'entregue', 'credito', 89.90),
(2, 2, '2025-05-02', 'entregue', 'debito', 129.90),
(3, 3, NULL, 'pendente', 'boleto', 200.00),
(4, 4, NULL, 'processando', 'pix', 99.90),
(5, 5, NULL, 'enviado', 'transferencia', 75.00),
(6, 6, '2025-05-03', 'entregue', 'credito', 150.00),
(7, 7, NULL, 'pendente', 'pix', 39.90),
(8, 8, NULL, 'cancelado', 'boleto', 0.00),
(9, 9, NULL, 'processando', 'credito', 79.90),
(10, 10, '2025-05-05', 'entregue', 'debito', 99.90);

-- 9. Tabela: Item Pedido

INSERT INTO ITEM_PEDIDO (id_pedido, numero_serie, preco_unitario, quantidade, desconto) VALUES
(1, 1, 59.90, 1, 0),
(1, 2, 30.00, 1, 0),
(2, 3, 89.50, 1, 0),
(3, 4, 42.00, 2, 10),
(4, 5, 99.90, 1, 5),
(5, 6, 35.00, 1, 0),
(6, 7, 64.90, 1, 0),
(7, 8, 29.90, 1, 0),
(9, 9, 110.00, 1, 0),
(10, 10, 49.90, 1, 0);

-- 10. Tabela: Área de Conhecimento

INSERT INTO AREA_CONHECIMENTO (codigo, descricao) VALUES
('TI', 'Tecnologia da Informação'),
('HIS', 'História'),
('LIT', 'Literatura'),
('INF', 'Infantil'),
('JRN', 'Jornalismo'),
('POE', 'Poesia'),
('EDU', 'Educação'),
('FIC', 'Ficção'),
('ADM', 'Administração'),
('DIRE', 'Direito');

-- 11. Tabela: Palavra-Chave

INSERT INTO PALAVRA_CHAVE (codigo, descricao) VALUES
('NET', 'Internet'),
('ROM', 'Romance'),
('POL', 'Política'),
('CRI', 'Crítica'),
('INF', 'Infantil'),
('POE', 'Poesia'),
('HIS', 'História'),
('FUT', 'Futuro'),
('TEC', 'Tecnologia'),
('EDU', 'Educação');

-- 12. Tabela: Livro Área

INSERT INTO LIVRO_AREA (isbn, id_area) VALUES
('9780000000001', 1),
('9780000000001', 8),
('9780000000002', 2),
('9780000000003', 3),
('9780000000004', 1),
('9780000000005', 4),
('9780000000006', 3),
('9780000000007', 6),
('9780000000008', 1),
('9780000000009', 7);

-- 13. Tabela: Livro Palavra

INSERT INTO LIVRO_PALAVRA (isbn, id_palavra) VALUES
('9780000000001', 8),
('9780000000001', 9),
('9780000000002', 7),
('9780000000003', 2),
('9780000000004', 10),
('9780000000005', 5),
('9780000000006', 4),
('9780000000007', 6),
('9780000000008', 1),
('9780000000009', 10);

-- 14. Histórico Preço

INSERT INTO HISTORICO_PRECO (isbn, preco_antigo, preco_novo, id_funcionario, motivo) VALUES
('9780000000001', 49.90, 59.90, 1, 'Reajuste anual'),
('9780000000002', 79.90, 89.50, 2, 'Aumento do custo de impressão'),
('9780000000003', 39.90, 42.00, 3, 'Demanda alta'),
('9780000000004', 89.90, 99.90, 4, 'Nova edição com conteúdo atualizado'),
('9780000000005', 29.90, 35.00, 5, 'Campanha de marketing'),
('9780000000006', 59.90, 64.90, 6, 'Correção de tabela'),
('9780000000007', 24.90, 29.90, 7, 'Inflação acumulada'),
('9780000000008', 99.90, 110.00, 8, 'Nova versão ampliada'),
('9780000000009', 44.90, 49.90, 9, 'Custos logísticos'),
('9780000000010', 69.90, 79.90, 10, 'Alta procura após lançamento');

# 3.1.5 Comandos UPDATE (10 exemplos):

-- Atualiza o telefone de um cliente
UPDATE CLIENTE SET telefone = '11999999999' WHERE id_cliente = 1;

-- Atualiza o estado de um exemplar emprestado
UPDATE EXEMPLAR SET estado = 'disponivel' WHERE numero_serie = 2;

-- Atualiza o salário de um funcionário
UPDATE FUNCIONARIO SET salario = salario * 1.1 WHERE id_funcionario = 4;

-- Atualiza o status de um pedido
UPDATE PEDIDO SET status = 'entregue', data_entrega = CURRENT_DATE WHERE id_pedido = 4;

-- Atualiza a descrição de uma área de conhecimento
UPDATE AREA_CONHECIMENTO SET descricao = 'Tecnologia e Inovação' WHERE codigo = 'TI';

-- Atualiza o título de um livro
UPDATE LIVRO SET titulo = 'O Futuro do Amanhã - Edição Especial' WHERE isbn = '9780000000001';

-- Atualiza a forma de pagamento de um pedido
UPDATE PEDIDO SET forma_pagamento = 'pix' WHERE id_pedido = 2;

-- Atualiza a quantidade de um item de pedido
UPDATE ITEM_PEDIDO SET quantidade = 2 WHERE id_pedido = 4 AND numero_serie = 5;

-- Atualiza o preço sugerido de um livro
UPDATE LIVRO SET preco_sugerido = 65.00 WHERE isbn = '9780000000006';

-- Marca funcionário como inativo
UPDATE FUNCIONARIO SET ativo = FALSE WHERE id_funcionario = 10;

# 3.1.6 Comandos DELETE (10 exemplos):

-- Remove palavra-chave associada ao livro perdido
DELETE FROM LIVRO_PALAVRA WHERE isbn = '9780000000007';

-- Remove área de conhecimento de um livro
DELETE FROM LIVRO_AREA WHERE isbn = '9780000000006' AND id_area = 3;

-- Remove item de pedido cancelado
DELETE FROM ITEM_PEDIDO WHERE id_pedido = 8;

-- Remove exemplar perdido
DELETE FROM EXEMPLAR WHERE estado = 'perdido';

-- Remove cliente duplicado (exemplo fictício)
DELETE FROM CLIENTE WHERE id_cliente = 10;

-- Remove histórico de preço de um livro específico
DELETE FROM HISTORICO_PRECO WHERE isbn = '9780000000005';

-- Remove um funcionário desligado
DELETE FROM FUNCIONARIO WHERE id_funcionario = 9;

-- Remove um autor com falecimento e sem livros
DELETE FROM AUTOR WHERE id_autor = 6;

-- Remove uma área de conhecimento obsoleta
DELETE FROM AREA_CONHECIMENTO WHERE codigo = 'ADM';

-- Remove palavra-chave que não está mais em uso
DELETE FROM PALAVRA_CHAVE WHERE codigo = 'CRI';

# 3.1.7 Relatórios e Consultas (DQL)

-- 1. Listar todos os livros com nome do autor e nome da editora

SELECT L.titulo, A.nome AS autor, E.nome AS editora
FROM LIVRO L
JOIN AUTOR A ON L.id_autor = A.id_autor
JOIN EDITORA E ON L.id_editora = E.id_editora;
-- 2. Listar os exemplares disponíveis com título e localização

SELECT E.numero_serie, L.titulo, E.localizacao
FROM EXEMPLAR E
JOIN LIVRO L ON E.isbn = L.isbn
WHERE E.estado = 'disponivel';

-- 3. Clientes que fizeram pedidos com mais de R$100

SELECT C.nome, P.valor_total
FROM CLIENTE C
JOIN PEDIDO P ON C.id_cliente = P.id_cliente
WHERE P.valor_total > 100;

-- 4. Listar livros com mais de uma área de conhecimento

SELECT L.titulo, COUNT(LA.id_area) AS qtd_areas
FROM LIVRO L
JOIN LIVRO_AREA LA ON L.isbn = LA.isbn
GROUP BY L.titulo
HAVING COUNT(LA.id_area) > 1;

-- 5. Exibir os livros mais caros (top 5)

SELECT titulo, preco_sugerido
FROM LIVRO
ORDER BY preco_sugerido DESC
LIMIT 5;

-- 6. Exibir livros com palavras-chave "tecnologia" ou "educação"

SELECT L.titulo, PK.descricao
FROM LIVRO L
JOIN LIVRO_PALAVRA LP ON L.isbn = LP.isbn
JOIN PALAVRA_CHAVE PK ON LP.id_palavra = PK.id_palavra
WHERE PK.descricao IN ('Tecnologia', 'Educação');

-- 7. Total de livros por editora

SELECT E.nome, COUNT(L.isbn) AS total_livros
FROM EDITORA E
LEFT JOIN LIVRO L ON E.id_editora = L.id_editora
GROUP BY E.nome;

-- 8. Listar os pedidos entregues com nome do cliente e data

SELECT C.nome, P.data_entrega
FROM PEDIDO P
JOIN CLIENTE C ON P.id_cliente = C.id_cliente
WHERE P.status = 'entregue';

-- 9. Buscar todos os livros publicados após 2020

SELECT titulo, data_publicacao
FROM LIVRO
WHERE data_publicacao > '2020-01-01';

-- 10. Relatório de funcionários por departamento

SELECT D.nome AS departamento, COUNT(F.id_funcionario) AS total_funcionarios
FROM DEPARTAMENTO D
LEFT JOIN FUNCIONARIO F ON D.id_departamento = F.id_departamento
GROUP BY D.nome;

-- 11. Valor médio dos pedidos por forma de pagamento

SELECT forma_pagamento, AVG(valor_total) AS media
FROM PEDIDO
GROUP BY forma_pagamento;

-- 12. Clientes que ainda não fizeram pedidos

SELECT nome
FROM CLIENTE
WHERE id_cliente NOT IN (SELECT id_cliente FROM PEDIDO);

-- 13. Livros com alterações de preço

SELECT L.titulo, HP.preco_antigo, HP.preco_novo, HP.data_alteracao
FROM HISTORICO_PRECO HP
JOIN LIVRO L ON HP.isbn = L.isbn;

-- 14. Top 3 autores com mais livros publicados

SELECT A.nome, COUNT(L.isbn) AS qtd_livros
FROM AUTOR A
JOIN LIVRO L ON A.id_autor = L.id_autor
GROUP BY A.nome
ORDER BY qtd_livros DESC
LIMIT 3;

-- 15. Exemplares emprestados com nome do livro

SELECT E.numero_serie, L.titulo
FROM EXEMPLAR E
JOIN LIVRO L ON E.isbn = L.isbn
WHERE E.estado = 'emprestado';

-- 16. Pedidos com mais de um item

SELECT id_pedido, COUNT(*) AS qtd_itens
FROM ITEM_PEDIDO
GROUP BY id_pedido
HAVING COUNT(*) > 1;

-- 17. Listar livros sem área de conhecimento vinculada

SELECT titulo
FROM LIVRO
WHERE isbn NOT IN (SELECT isbn FROM LIVRO_AREA);

-- 18. Listar livros e quantas palavras-chave possuem

SELECT L.titulo, COUNT(LP.id_palavra) AS qtd_palavras
FROM LIVRO L
LEFT JOIN LIVRO_PALAVRA LP ON L.isbn = LP.isbn
GROUP BY L.titulo;

-- 19. Funcionários com salário acima da média

SELECT nome, salario
FROM FUNCIONARIO
WHERE salario > (SELECT AVG(salario) FROM FUNCIONARIO);

-- 20. Palavras-chave mais associadas a livros (top 3)

SELECT P.descricao, COUNT(LP.isbn) AS ocorrencias
FROM PALAVRA_CHAVE P
JOIN LIVRO_PALAVRA LP ON P.id_palavra = LP.id_palavra
GROUP BY P.descricao
ORDER BY ocorrencias DESC
LIMIT 3;

# 3.1.8 Criação de Views (mínimo 10):

-- 1. View: Livros com Autor e Editora

CREATE VIEW vw_livros_autores_editoras AS
SELECT 
    L.isbn,
    L.titulo,
    A.nome AS autor,
    E.nome AS editora,
    L.data_publicacao,
    L.preco_sugerido
FROM LIVRO L
JOIN AUTOR A ON L.id_autor = A.id_autor
JOIN EDITORA E ON L.id_editora = E.id_editora;

-- 2. View: Exemplares Disponíveis

CREATE VIEW vw_exemplares_disponiveis AS
SELECT 
    E.numero_serie,
    L.titulo,
    E.localizacao,
    E.data_aquisicao
FROM EXEMPLAR E
JOIN LIVRO L ON E.isbn = L.isbn
WHERE E.estado = 'disponivel';

-- 3. View: Pedidos Entregues

CREATE VIEW vw_pedidos_entregues AS
SELECT 
    P.id_pedido,
    C.nome AS cliente,
    P.data_pedido,
    P.data_entrega,
    P.valor_total,
    P.forma_pagamento
FROM PEDIDO P
JOIN CLIENTE C ON P.id_cliente = C.id_cliente
WHERE P.status = 'entregue';

-- 4. View: Total de Livros por Editora

CREATE VIEW vw_total_livros_por_editora AS
SELECT 
    E.nome AS editora,
    COUNT(L.isbn) AS total_livros
FROM EDITORA E
LEFT JOIN LIVRO L ON E.id_editora = L.id_editora
GROUP BY E.nome;

-- 5. View: Funcionários por Departamento

CREATE VIEW vw_funcionarios_departamento AS
SELECT 
    D.nome AS departamento,
    F.nome AS funcionario,
    F.cargo,
    F.salario
FROM FUNCIONARIO F
JOIN DEPARTAMENTO D ON F.id_departamento = D.id_departamento;

-- 6. View: Livros com Múltiplas Áreas

CREATE VIEW vw_livros_multiplas_areas AS
SELECT 
    L.titulo,
    COUNT(LA.id_area) AS qtd_areas
FROM LIVRO L
JOIN LIVRO_AREA LA ON L.isbn = LA.isbn
GROUP BY L.titulo
HAVING COUNT(LA.id_area) > 1;

-- 7. View: Histórico de Preços

CREATE VIEW vw_historico_precos AS
SELECT 
    L.titulo,
    HP.preco_antigo,
    HP.preco_novo,
    HP.data_alteracao,
    F.nome AS alterado_por
FROM HISTORICO_PRECO HP
JOIN LIVRO L ON HP.isbn = L.isbn
LEFT JOIN FUNCIONARIO F ON HP.id_funcionario = F.id_funcionario;

-- 8. View: Clientes Sem Pedidos

CREATE VIEW vw_clientes_sem_pedidos AS
SELECT 
    C.id_cliente,
    C.nome,
    C.email
FROM CLIENTE C
WHERE NOT EXISTS (
    SELECT 1
    FROM PEDIDO P
    WHERE P.id_cliente = C.id_cliente
);

-- 9. View: Livros com Palavras-chave

CREATE VIEW vw_livros_palavras_chave AS
SELECT 
    L.titulo,
    PK.descricao AS palavra_chave
FROM LIVRO L
JOIN LIVRO_PALAVRA LP ON L.isbn = LP.isbn
JOIN PALAVRA_CHAVE PK ON LP.id_palavra = PK.id_palavra;

-- 10. View: Valor Médio por Forma de Pagamento

CREATE VIEW vw_media_valor_pagamento AS
SELECT 
    forma_pagamento,
    AVG(valor_total) AS media_valor
FROM PEDIDO
GROUP BY forma_pagamento;
