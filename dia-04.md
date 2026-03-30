# aula 02
replicaset - principal função é garantir que as réplicas de pods estão no número desejado. criado automaticamente junto da criação do deployment. Não é recomendado criar replicaset na mão
cada vez que o deployment é atualizado, é criado novo replicaset. novo deployment identifica replicaset atual e replicaset anterior. Com isso o rollback para versão anterior fica mais simples, pois o deployment anterior já vai apontar para o replicaset anterior
