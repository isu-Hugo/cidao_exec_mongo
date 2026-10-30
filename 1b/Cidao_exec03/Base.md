use empresa_tech;

db.funcionarios.drop();

db.funcionarios.insertMany([
  { nome: "Ana Silva",            cargo: "Desenvolvedora Frontend",  departamento: "Tecnologia", salario: 8500,  ativo: true,
    habilidades: ["JavaScript", "React", "CSS"],
    contato: { email: "ana.silva@empresa.com",       telefone: "11-98765-4321" } },

  { nome: "Bruno Costa",          cargo: "Desenvolvedor Backend",    departamento: "Tecnologia", salario: 9200,  ativo: true,
    habilidades: ["Java", "Spring", "MySQL"],
    contato: { email: "bruno.costa@empresa.com",     telefone: "11-97654-3210" } },

  { nome: "Carla Marques",        cargo: "Analista de BI",           departamento: "Dados",      salario: 7300,  ativo: true,
    habilidades: ["SQL", "PostgreSQL", "Power BI"],
    contato: { email: "carla.marques@empresa.com",   telefone: "21-96543-2109" } },

  { nome: "Andre Souza",          cargo: "Desenvolvedor Fullstack",  departamento: "Tecnologia", salario: 11000, ativo: true,
    habilidades: ["JavaScript", "Node.js", "React", "MongoDB"],
    contato: { email: "andre.souza@empresa.com",     telefone: "11-95432-1098" } },

  { nome: "Daniela Prado",        cargo: "Gerente de Projetos",      departamento: "Projetos",   salario: 14500, ativo: true,
    habilidades: ["Jira", "Scrum", "Kanban"],
    contato: { email: "daniela.prado@empresa.com",   telefone: "11-94321-0987" } },

  // e-mail fora do dominio da empresa (gmail)
  { nome: "Eduardo Lima",         cargo: "Analista de Suporte",      departamento: "Suporte",    salario: 4200,  ativo: false,
    habilidades: ["Windows Server", "Zendesk"],
    contato: { email: "eduardo.lima@gmail.com",      telefone: "31-93210-9876" } },

  // telefone sem hifens
  { nome: "Fernanda Silva",       cargo: "Desenvolvedora Backend",   departamento: "Tecnologia", salario: 9800,  ativo: true,
    habilidades: ["Python", "Django", "PostgreSQL"],
    contato: { email: "fernanda.silva@empresa.com",  telefone: "1187654321" } },

  // telefone fixo (nao bate o padrao de celular DD-9XXXX-XXXX)
  { nome: "Gabriel Marques Costa", cargo: "Analista de Dados",       departamento: "Dados",      salario: 6900,  ativo: true,
    habilidades: ["Python", "SQL", "Pandas"],
    contato: { email: "gabriel.costa@empresa.com",   telefone: "11-3456-7890" } },

  { nome: "Helena Rocha",         cargo: "Designer UX",              departamento: "Produto",    salario: 7800,  ativo: true,
    habilidades: ["Figma", "CSS", "Design System"],
    contato: { email: "helena.rocha@empresa.com",    telefone: "41-92109-8765" } },

  // e-mail SEM arroba - erro de cadastro
  { nome: "Igor Nunes",           cargo: "Estagiario de TI",         departamento: "Tecnologia", salario: 2100,  ativo: true,
    habilidades: ["Java", "Git"],
    contato: { email: "igor.nunes-empresa.com",      telefone: "11-91098-7654" } },

  { nome: "Juliana Alves",        cargo: "Gerente de RH",            departamento: "RH",         salario: 12000, ativo: true,
    habilidades: ["Gestao de Pessoas", "Excel"],
    contato: { email: "juliana.alves@empresa.com",   telefone: "11-90987-6543" } },

  // nome com uma palavra so + e-mail fora do padrao nome.sobrenome
  { nome: "Marcos",               cargo: "Analista Financeiro",      departamento: "Financeiro", salario: 6500,  ativo: false,
    habilidades: ["Excel", "SAP"],
    contato: { email: "marcos@empresa.com",          telefone: "11-98876-5544" } },

  { nome: "Ana Paula Ribeiro",    cargo: "Desenvolvedora Mobile",    departamento: "Tecnologia", salario: 8900,  ativo: true,
    habilidades: ["Kotlin", "Java", "Firebase"],
    contato: { email: "anapaula.ribeiro@empresa.com", telefone: "51-98877-6655" } },

  { nome: "Otavio Marques",       cargo: "Arquiteto de Software",    departamento: "Tecnologia", salario: 16000, ativo: true,
    habilidades: ["Java", "Kubernetes", "MongoDB"],
    contato: { email: "otavio.marques@empresa.com",  telefone: "11-97766-5544" } },

  // dominio com letra maiuscula - so aparece com a flag "i"
  { nome: "Patricia Costa",       cargo: "Analista de Marketing",    departamento: "Marketing",  salario: 5600,  ativo: true,
    habilidades: ["SEO", "Google Analytics"],
    contato: { email: "patricia.costa@Empresa.com",  telefone: "11-96655-4433" } },

  // SEM o campo contato.email - a pegadinha do $not
  { nome: "Rafael Teixeira",      cargo: "Analista de Infra",        departamento: "Tecnologia", salario: 8100,  ativo: true,
    habilidades: ["Linux", "Docker", "AWS"],
    contato: { telefone: "11-95544-3322" } }
]);