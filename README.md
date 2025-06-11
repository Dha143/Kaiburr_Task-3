1 Project Bootstrapping

Start with a Vite-based React+TS template:

```bash
npm create vite@latest task-ui --template react-ts
cd task-ui
npm install antd axios dayjs @ant-design/icons
npm install less less-plugin-inline-import --save-dev
```

Import Ant Design styles in `main.tsx`:

```ts
import 'antd/dist/reset.css';
import './index.less';
```

Configure Less if theming is needed.

---

## 2. 🧭 UI Structure

| Component        | Functionality                                                |
| ---------------- | ------------------------------------------------------------ |
| `TaskList`       | Displays tasks in a table with search and action buttons.    |
| `TaskFormModal`  | Modal form to create/edit tasks using `<Form>` + validation. |
| `ExecutionModal` | Shows output history with timestamps.                        |
| `App`            | Holds global state and API interactions via `axios`.         |

---

## 3. ✅ Usability & Accessibility Highlights

### – Accessible Form Controls

Use `<Form.Item label="Name">` tied to inputs for clear labeling. Leverage ARIA support from AntD ([dejanvasic.wordpress.com][1], [dev.to][2], [leandroaps.medium.com][3]).

### – Keyboard-Friendly UI

AntD components support keyboard nav and focus management . Ensure modals trap focus and buttons are focusable.

### – Clear Validation & Feedback

Utilize AntD's `rules` prop for validation. Show loading indicators on submitting actions. Offer success/error messages via `message`.

---

## 4. 🧩 Core Implementation Snippets

### App.tsx – Central UI Controller

```tsx
const App: FC = () => {
  const [tasks, setTasks] = useState<Task[]>([]);
  const [loading, setLoading] = useState(false);
  const [formVisible, setFormVisible] = useState(false);
  const [executionModal, setExecutionModal] = useState<Task | null>(null);

  const fetchTasks = async (name?: string) => {
    setLoading(true);
    const res = await axios.get<Task[]>('/tasks', { params: name ? { name } : {} });
    setTasks(res.data);
    setLoading(false);
  };

  const deleteTask = async (id: string) => {
    await axios.delete(`/tasks/${id}`);
    message.success('Deleted');
    fetchTasks();
  };

  const runTask = async (id: string) => {
    await axios.put<TaskExecution>(`/tasks/${id}/run`);
    fetchTasks();
  };

  useEffect(() => { fetchTasks(); }, []);
```

### TaskFormModal.tsx

```tsx
<Form form={form} layout="vertical" onFinish={onFinish}>
  <Form.Item name="id" label="ID" rules={[{ required: true }]}>
    <Input aria-label="Task ID" />
  </Form.Item>
  <Form.Item name="command" label="Command"
    rules={[{ required: true, pattern: /^[a-zA-Z0-9_\- ./]+$/, message: 'Unsafe command' }]}>
    <Input aria-label="Shell command" />
  </Form.Item>
  <Button htmlType="submit" type="primary">Save</Button>
</Form>
```

### TaskList & Execution Modal

```tsx
<Table dataSource={tasks} rowKey="id" loading={loading}>
  <Column title="Name" dataIndex="name" />
  <Column title="Actions"
    render={(text, task) => (
      <>
        <Button onClick={() => runTask(task.id)} icon={<PlayCircleOutlined />} />
        <Button onClick={() => deleteTask(task.id)} danger icon={<DeleteOutlined />} />
        <Button onClick={() => setExecutionModal(task)} icon={<EyeOutlined />} />
      </>
    )}
  />
</Table>

<Modal visible={!!executionModal} onCancel={() => setExecutionModal(null)} footer={null}>
  {executionModal?.taskExecutions.map(exec => (
    <div key={+exec.startTime}>
      <strong>{dayjs(exec.startTime).format('YYYY-MM-DD HH:mm:ss')}</strong>
      <pre>{exec.output}</pre>
    </div>
  ))}
</Modal>
```

---

