# Tutorial-relaciones-sequelize

Hola chiques, cómo va? Traduzco este tutorial más que nada para que se me fijen los conocimientos de este tema tan complicado jaja. 
Pero si les puede servir a ustedes mejor!

También lo hago porque me dí cuenta de que no existen muchos ejemplos concretos de cómo hacer determinadas cosas con Sequelize, y menos en español.

Este artículo contiene detalles de cómo implementar todos los tipos de relaciones posibles en Sequelize.

Bien, empecemos entonces!

## Los tres tipos de relaciones

Tomaremos como un ejemplo 3 modelos.
-Empresa/Company
-Empleado/Employee
-Día de Trabajo/Working Day

Empleado tiene una Empresa(belongsTo) 1:1
Empresa tiene muchos Empleados(hasMany) 1:n
Empleado tiene muchos Días de Trabajo y Días de trabajo tiene muchos Empleados

Para crear una relación de muchos a muchos, necesitaremos una tabla intermedia que vamos a llamar WorkingDaysEmployee.

## Express Generator y Sequelize CLI

Esto nos va a permitir iniciar un proyecto con una estructura inicial cómoda.

Si no está hecho ya, instalar globalmente express-generator y sequelize-cli así:

```
  npm install express-generator -g
  npm install -g sequelize-cli
```


Esto nos va a generar una nueva App gracias a Express y nos va a instalar las librerías necesarias:

```
  express app-relaciones-practica --no-view
  cd app-relaciones-practica
  npm install --save sequelize pg pg-hstore   //O el dialecto que prefieran ustedes, en este caso usamos PostgreSQL
  sequelize init
  
  Este último comando nos va a crear 3 carpetas(Models, Migrations, y Seeders) y un archivo 'config.json en el directorio '/config'
```

Necesitamos modificar este archivo con tus credenciales para el servidor local de postgres así:

```
//config.json

{
  "development": {
    "username": "root",
    "password": "My_mysql_root_p@ssw0rd",
    "database": "nodejs_relationship_demo",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "test": {
    "username": "root",
    "password": null,
    "database": "database_test",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "production": {
    "username": "root",
    "password": null,
    "database": "database_production",
    "host": "127.0.0.1",
    "dialect": "mysql"
  }
}
```

Después de la configuración podés ejecutar los siguientes comandos:

```
sequelize db:create   //Para crear una base de datos nueva llamada:
relaciones_demo

```
### Generación de Modelos

Ahora, vamos a crear todos los modelos que vamos a necesitar:

```
sequelize model:generate --name Company --attributes name:STRING
sequelize model:generate --name User --attributes email:STRING,firstName:STRING,lastName:STRING,companyId:INTEGER
sequelize model:generate --name WorkingDay --attributes weekDay:STRING,workingDate:DATE,isWorking:BOOLEAN
sequelize model:generate --name UsersWorkingDay --attributes userId:INTEGER,workingDayId:INTEGER
```

UsersWorkingDay será usado para unir(join) User y WorkingDay (Relación de muchos a muchos)
Ahora que generamos las plantillas necesarias, vamos a plantear las realciones en los archivos de migración añadiendo una
clave(key) de referencia en todas las claves foráneas(foreign keys).
También le añadimos a cada clave foránea un AllowNull false para prevenir creación si el modelo no está enlazado.

```
// 20181128093807-create-company.js 
'use strict';
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('Companies', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      name: {
        type: Sequelize.STRING
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable('Companies');
  }
};
```


```
//20181128093808-create-user.js
'use strict';
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('Users', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      email: {
        type: Sequelize.STRING
      },
      firstName: {
        type: Sequelize.STRING
      },
      lastName: {
        type: Sequelize.STRING
      },
      companyId: {
        type: Sequelize.INTEGER,
        allowNull: false,
        references: {         // User belongsTo Company 1:1
          model: 'Companies',
          key: 'id'
        }
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable('Users');
  }
};

```

```
'use strict';
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('WorkingDays', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      weekDay: {
        type: Sequelize.STRING
      },
      workingDate: {
        type: Sequelize.DATE
      },
      isWorking: {
        type: Sequelize.BOOLEAN
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable('WorkingDays');
  }
};
```

```
//20181128093809-create-users-working-day.js 
'use strict';
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('UsersWorkingDays', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      userId: {
        type: Sequelize.INTEGER,
        allowNull: false,
        references: {         // User hasMany WorkingDays n:n
          model: 'Users',
          key: 'id'
        }
      },
      workingDayId: {
        type: Sequelize.INTEGER,
        allowNull: false,
        references: {         // WorkingDays hasMany Users n:n
          model: 'WorkingDays',
          key: 'id'
        }
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  down: (queryInterface, Sequelize) => {
    return queryInterface.dropTable('UsersWorkingDays');
  }
};
```
