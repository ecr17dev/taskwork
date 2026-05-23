Angular **no debería conectarse directo a SQLite**. Lo profesional es que Angular hable con una API, y la API tenga services/repositories que administren la base de datos.

---

# Proyecto: TaskFlow Pro con Angular + SQLite

## 1. Autenticación y usuarios

### Funcionalidades

- Login de usuario.
- Logout.
- Registro opcional.
- Roles:
  - `admin`
  - `manager`
  - `developer`

- Protección de rutas.
- Usuario actual persistido.
- Permisos por rol.

### Pantallas

```txt
/login
/register
/profile
/admin/users
```

### Tablas SQLite

```sql
users
- id
- name
- email
- password_hash
- role
- avatar_url
- created_at
- updated_at
```

### Services Angular

```ts
AuthService -
  login(credentials) -
  logout() -
  getCurrentUser() -
  isAuthenticated() -
  hasRole(role);
```

```ts
UserService -
  getUsers() -
  getUserById(id) -
  createUser(data) -
  updateUser(id, data) -
  deleteUser(id);
```

### Services backend

```ts
AuthService -
  validateUser() -
  generateToken() -
  hashPassword() -
  comparePassword();
```

```ts
UserService
- findAll()
- findById()
- findByEmail()
- create()
- update()
- delete()
```

---

# 2. Gestión de proyectos

## Funcionalidades

- Crear proyectos.
- Editar proyectos.
- Archivar proyectos.
- Asignar usuarios a un proyecto.
- Ver progreso general.
- Ver tareas asociadas.
- Filtrar proyectos por estado.

### Estados del proyecto

```txt
active
paused
completed
archived
```

### Pantallas

```txt
/projects
/projects/new
/projects/:id
/projects/:id/settings
/projects/:id/members
```

### Tabla SQLite

```sql
projects
- id
- name
- description
- status
- owner_id
- start_date
- end_date
- created_at
- updated_at
```

```sql
project_members
- id
- project_id
- user_id
- role
- created_at
```

### Services Angular

```ts
ProjectService -
  getProjects(filters) -
  getProjectById(id) -
  createProject(data) -
  updateProject(id, data) -
  archiveProject(id) -
  deleteProject(id);
```

```ts
ProjectMemberService -
  getProjectMembers(projectId) -
  addMember(projectId, userId, role) -
  updateMemberRole(projectId, userId, role) -
  removeMember(projectId, userId);
```

### Services backend

```ts
ProjectService
- findAll()
- findById()
- create()
- update()
- archive()
- delete()
```

```ts
ProjectMemberService -
  findMembersByProject() -
  addMember() -
  changeRole() -
  removeMember();
```

---

# 3. Gestión avanzada de tareas

Esta sería la parte central del proyecto.

## Funcionalidades

- Crear tarea.
- Editar tarea.
- Eliminar tarea.
- Asignar responsable.
- Asociar tarea a proyecto.
- Definir prioridad.
- Definir estado.
- Definir fecha límite.
- Agregar descripción.
- Marcar tarea como completada.
- Ver historial de cambios.

### Estados de tarea

```txt
backlog
todo
in_progress
review
done
blocked
```

### Prioridades

```txt
low
medium
high
urgent
```

### Pantallas

```txt
/tasks
/tasks/new
/tasks/:id
/projects/:projectId/tasks
/projects/:projectId/tasks/:taskId
```

### Tabla SQLite

```sql
tasks
- id
- project_id
- title
- description
- status
- priority
- assignee_id
- reporter_id
- due_date
- completed_at
- created_at
- updated_at
```

### Services Angular

```ts
TaskService -
  getTasks(filters) -
  getTaskById(id) -
  createTask(data) -
  updateTask(id, data) -
  deleteTask(id) -
  changeStatus(id, status) -
  assignUser(id, userId) -
  completeTask(id);
```

### Services backend

```ts
TaskService
- findAll(filters)
- findById()
- create()
- update()
- delete()
- changeStatus()
- assignUser()
- complete()
```

---

# 4. Kanban board

## Funcionalidades

- Ver tareas agrupadas por estado.
- Mover tareas con drag & drop.
- Cambiar estado automáticamente.
- Ordenar tareas dentro de una columna.
- Filtrar por responsable, prioridad o proyecto.
- Mostrar cantidad de tareas por columna.

### Pantalla

```txt
/projects/:projectId/kanban
```

### Tabla adicional para orden

```sql
task_positions
- id
- task_id
- project_id
- status
- position
- updated_at
```

También podrías guardar `position` directo en `tasks`, así:

```sql
tasks
- id
- status
- position
```

Más simple para empezar.

### Services Angular

```ts
KanbanService -
  getBoard(projectId) -
  moveTask(taskId, newStatus, newPosition) -
  reorderTasks(projectId, status, orderedTaskIds);
```

### Services backend

```ts
KanbanService - getBoardByProject() - moveTask() - reorderColumn();
```

---

# 5. Comentarios en tareas

## Funcionalidades

- Agregar comentario a una tarea.
- Editar comentario propio.
- Eliminar comentario propio.
- Ver comentarios ordenados por fecha.
- Mostrar autor del comentario.
- Soporte para menciones opcional.

### Tabla SQLite

```sql
task_comments
- id
- task_id
- user_id
- content
- created_at
- updated_at
```

### Services Angular

```ts
CommentService -
  getCommentsByTask(taskId) -
  addComment(taskId, content) -
  updateComment(commentId, content) -
  deleteComment(commentId);
```

### Services backend

```ts
CommentService
- findByTask()
- create()
- update()
- delete()
```

---

# 6. Historial de actividad

Esta funcionalidad te enseña diseño más profesional.

## Funcionalidades

Registrar eventos como:

```txt
Tarea creada
Estado cambiado
Prioridad modificada
Responsable asignado
Comentario agregado
Fecha límite cambiada
Tarea completada
```

### Pantalla

```txt
/tasks/:id/activity
```

O dentro del detalle de tarea como una pestaña.

### Tabla SQLite

```sql
activity_logs
- id
- entity_type
- entity_id
- action
- old_value
- new_value
- user_id
- created_at
```

Ejemplo:

```txt
entity_type: task
entity_id: 15
action: status_changed
old_value: todo
new_value: in_progress
user_id: 3
```

### Services Angular

```ts
ActivityService - getActivityByTask(taskId) - getActivityByProject(projectId);
```

### Services backend

```ts
ActivityLogService - log() - findByEntity() - findByProject();
```

---

# 7. Dashboard con métricas

## Funcionalidades

- Total de proyectos activos.
- Total de tareas abiertas.
- Total de tareas completadas.
- Tareas atrasadas.
- Tareas por prioridad.
- Tareas por estado.
- Productividad por usuario.
- Última actividad.

### Pantalla

```txt
/dashboard
```

### Consultas útiles en SQLite

```sql
SELECT status, COUNT(*)
FROM tasks
GROUP BY status;
```

```sql
SELECT priority, COUNT(*)
FROM tasks
GROUP BY priority;
```

```sql
SELECT COUNT(*)
FROM tasks
WHERE due_date < datetime('now')
AND status != 'done';
```

### Services Angular

```ts
DashboardService -
  getSummary() -
  getTasksByStatus() -
  getTasksByPriority() -
  getOverdueTasks() -
  getRecentActivity();
```

### Services backend

```ts
DashboardService -
  getSummary() -
  countTasksByStatus() -
  countTasksByPriority() -
  findOverdueTasks() -
  findRecentActivity();
```

---

# 8. Búsqueda y filtros avanzados

## Funcionalidades

Filtrar tareas por:

```txt
Proyecto
Estado
Prioridad
Responsable
Fecha límite
Texto
Creador
```

Ordenar por:

```txt
Fecha de creación
Fecha límite
Prioridad
Estado
Responsable
```

Paginación:

```txt
page
limit
total
```

### Endpoint ejemplo

```txt
GET /api/tasks?status=in_progress&priority=high&page=1&limit=10
```

### Services Angular

```ts
TaskQueryService - buildTaskQuery(filters) - parseTaskResponse(response);
```

O puedes mantenerlo dentro de `TaskService`.

### Backend

```ts
TaskService.findAll({
  status,
  priority,
  assigneeId,
  projectId,
  search,
  page,
  limit,
  sortBy,
  sortDirection,
});
```

---

# 9. Adjuntos simulados o reales

Para empezar, puedes guardar solo metadata.

## Funcionalidades

- Subir archivo a una tarea.
- Listar archivos de una tarea.
- Descargar archivo.
- Eliminar archivo.
- Mostrar tamaño, tipo y fecha.

### Tabla SQLite

```sql
task_attachments
- id
- task_id
- file_name
- file_path
- mime_type
- size
- uploaded_by
- created_at
```

### Services Angular

```ts
AttachmentService -
  getAttachments(taskId) -
  uploadAttachment(taskId, file) -
  deleteAttachment(attachmentId) -
  downloadAttachment(attachmentId);
```

### Services backend

```ts
AttachmentService
- findByTask()
- upload()
- delete()
- download()
```

---

# 10. Notificaciones internas

No necesitas push real al inicio. Puedes hacer notificaciones dentro de la app.

## Funcionalidades

Crear notificación cuando:

```txt
Te asignan una tarea
Comentan una tarea tuya
Cambian la fecha límite
Una tarea vence pronto
Te agregan a un proyecto
```

### Tabla SQLite

```sql
notifications
- id
- user_id
- title
- message
- type
- is_read
- entity_type
- entity_id
- created_at
```

### Services Angular

```ts
NotificationService -
  getMyNotifications() -
  markAsRead(id) -
  markAllAsRead() -
  deleteNotification(id);
```

### Services backend

```ts
NotificationService
- create()
- findByUser()
- markAsRead()
- markAllAsRead()
- delete()
```

---

# Arquitectura Angular recomendada

```txt
src/app/
  core/
    auth/
      auth.service.ts
      auth.guard.ts
      role.guard.ts
      token.interceptor.ts

    api/
      api-client.service.ts

    layout/
      main-layout.component.ts
      sidebar.component.ts
      navbar.component.ts

  shared/
    components/
      button/
      modal/
      table/
      badge/
      confirm-dialog/

    pipes/
      status-label.pipe.ts
      priority-label.pipe.ts

    directives/
      has-role.directive.ts

  features/
    dashboard/
      dashboard-page.component.ts
      dashboard.service.ts

    projects/
      pages/
        project-list-page.component.ts
        project-detail-page.component.ts
        project-form-page.component.ts
      components/
        project-card.component.ts
        project-form.component.ts
        project-members.component.ts
      services/
        project.service.ts
        project-member.service.ts

    tasks/
      pages/
        task-list-page.component.ts
        task-detail-page.component.ts
        task-form-page.component.ts
      components/
        task-table.component.ts
        task-form.component.ts
        task-comments.component.ts
        task-activity.component.ts
      services/
        task.service.ts
        comment.service.ts
        activity.service.ts

    kanban/
      kanban-board-page.component.ts
      kanban-column.component.ts
      kanban-card.component.ts
      kanban.service.ts

    notifications/
      notification-list.component.ts
      notification.service.ts
```

---

# Arquitectura backend recomendada

Puedes hacerlo con **NestJS** o **Express**. Para aprender de forma más profesional, recomiendo NestJS.

```txt
src/
  database/
    database.module.ts
    sqlite.service.ts

  auth/
    auth.controller.ts
    auth.service.ts
    auth.guard.ts

  users/
    user.controller.ts
    user.service.ts
    user.repository.ts

  projects/
    project.controller.ts
    project.service.ts
    project.repository.ts

  tasks/
    task.controller.ts
    task.service.ts
    task.repository.ts

  comments/
    comment.controller.ts
    comment.service.ts
    comment.repository.ts

  activity/
    activity.controller.ts
    activity.service.ts
    activity.repository.ts

  dashboard/
    dashboard.controller.ts
    dashboard.service.ts
```

---

# Capas recomendadas

## En Angular

```txt
Component
   ↓
Service
   ↓
HttpClient
   ↓
API Backend
```

Ejemplo:

```ts
@Component({...})
export class TaskListPageComponent {
  private taskService = inject(TaskService);

  tasks = signal<Task[]>([]);

  ngOnInit() {
    this.taskService.getTasks().subscribe(tasks => {
      this.tasks.set(tasks);
    });
  }
}
```

```ts
@Injectable({ providedIn: "root" })
export class TaskService {
  private http = inject(HttpClient);
  private apiUrl = "/api/tasks";

  getTasks(filters?: TaskFilters) {
    return this.http.get<Task[]>(this.apiUrl, { params: { ...filters } });
  }

  getTaskById(id: number) {
    return this.http.get<Task>(`${this.apiUrl}/${id}`);
  }

  createTask(data: CreateTaskDto) {
    return this.http.post<Task>(this.apiUrl, data);
  }

  updateTask(id: number, data: UpdateTaskDto) {
    return this.http.patch<Task>(`${this.apiUrl}/${id}`, data);
  }

  deleteTask(id: number) {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

---

## En backend

```txt
Controller
   ↓
Service
   ↓
Repository
   ↓
SQLite
```

Ejemplo conceptual:

```ts
@Controller("tasks")
export class TaskController {
  constructor(private readonly taskService: TaskService) {}

  @Get()
  findAll(@Query() filters: TaskFiltersDto) {
    return this.taskService.findAll(filters);
  }

  @Post()
  create(@Body() data: CreateTaskDto) {
    return this.taskService.create(data);
  }

  @Patch(":id")
  update(@Param("id") id: string, @Body() data: UpdateTaskDto) {
    return this.taskService.update(Number(id), data);
  }
}
```

```ts
@Injectable()
export class TaskService {
  constructor(
    private readonly taskRepository: TaskRepository,
    private readonly activityLogService: ActivityLogService,
  ) {}

  async create(data: CreateTaskDto) {
    const task = await this.taskRepository.create(data);

    await this.activityLogService.log({
      entityType: "task",
      entityId: task.id,
      action: "task_created",
      newValue: JSON.stringify(task),
      userId: data.reporterId,
    });

    return task;
  }
}
```

---

# Módulos funcionales finales

Yo lo dividiría así:

```txt
1. Auth
2. Users
3. Projects
4. Project Members
5. Tasks
6. Kanban
7. Comments
8. Activity Logs
9. Attachments
10. Notifications
11. Dashboard
12. Settings
```

---

# MVP recomendado

Para no hacerlo gigante desde el inicio, parte con esto:

## Semana 1: Base

```txt
Auth básico
Layout principal
SQLite configurado
Tabla users
Tabla projects
Tabla tasks
CRUD de proyectos
CRUD de tareas
```

## Semana 2: Funcionalidad real

```txt
Filtros de tareas
Detalle de tarea
Asignación de usuarios
Comentarios
Cambio de estado
Historial de actividad
```

## Semana 3: Angular más pro

```txt
Kanban drag & drop
Signals para estado local
Guards
Interceptors
Reactive Forms
Validaciones
Loading states
Error handling
```

## Semana 4: Nivel portfolio

```txt
Dashboard
Notificaciones internas
Roles/permisos
Testing
Deploy
Documentación
```

---

# Modelo mínimo de base de datos

Para empezar, usaría estas tablas:

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  email TEXT NOT NULL UNIQUE,
  password_hash TEXT NOT NULL,
  role TEXT NOT NULL DEFAULT 'developer',
  created_at TEXT DEFAULT CURRENT_TIMESTAMP,
  updated_at TEXT DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE projects (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  description TEXT,
  status TEXT NOT NULL DEFAULT 'active',
  owner_id INTEGER NOT NULL,
  created_at TEXT DEFAULT CURRENT_TIMESTAMP,
  updated_at TEXT DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (owner_id) REFERENCES users(id)
);

CREATE TABLE project_members (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  project_id INTEGER NOT NULL,
  user_id INTEGER NOT NULL,
  role TEXT NOT NULL DEFAULT 'member',
  created_at TEXT DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (project_id) REFERENCES projects(id),
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE tasks (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  project_id INTEGER NOT NULL,
  title TEXT NOT NULL,
  description TEXT,
  status TEXT NOT NULL DEFAULT 'todo',
  priority TEXT NOT NULL DEFAULT 'medium',
  assignee_id INTEGER,
  reporter_id INTEGER NOT NULL,
  due_date TEXT,
  position INTEGER DEFAULT 0,
  completed_at TEXT,
  created_at TEXT DEFAULT CURRENT_TIMESTAMP,
  updated_at TEXT DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (project_id) REFERENCES projects(id),
  FOREIGN KEY (assignee_id) REFERENCES users(id),
  FOREIGN KEY (reporter_id) REFERENCES users(id)
);

CREATE TABLE task_comments (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  task_id INTEGER NOT NULL,
  user_id INTEGER NOT NULL,
  content TEXT NOT NULL,
  created_at TEXT DEFAULT CURRENT_TIMESTAMP,
  updated_at TEXT DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (task_id) REFERENCES tasks(id),
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE TABLE activity_logs (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  entity_type TEXT NOT NULL,
  entity_id INTEGER NOT NULL,
  action TEXT NOT NULL,
  old_value TEXT,
  new_value TEXT,
  user_id INTEGER,
  created_at TEXT DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

---

# Orden de implementación sugerido

Yo lo haría en este orden:

```txt
1. Crear backend con SQLite
2. Crear schema de base de datos
3. Crear UserService
4. Crear AuthService
5. Crear ProjectService
6. Crear TaskService
7. Conectar Angular con HttpClient
8. Crear layout Angular
9. Crear CRUD de proyectos
10. Crear CRUD de tareas
11. Agregar filtros
12. Agregar Kanban
13. Agregar comentarios
14. Agregar activity logs
15. Agregar dashboard
```

---

# Objetivo profesional del proyecto

Al terminarlo, podrías decir que sabes trabajar con:

```txt
Angular con arquitectura por features
Services bien separados
HTTP Client
Guards
Interceptors
Reactive Forms
Signals
SQLite
API REST
DTOs
Validaciones
Relaciones entre tablas
Dashboard
Filtros
Paginación
Drag & drop
Roles y permisos
```
