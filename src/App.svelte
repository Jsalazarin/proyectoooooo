<script>
  import Counter from './lib/Counter.svelte'
   import Toastify from "toastify-js";
  import { onDestroy } from "svelte";

  let task = {
    title: "",
    description: "",
  };

  let tasks = [];
  let inputElement;

  let editStatus = false;
  let currentId = "";

  const unsub = onSnapshot(
    collection(db, "tasks"),
    (querySnapshot) => {
      tasks = querySnapshot.docs.map((doc) => ({
        ...doc.data(),
        id: doc.id,
      }));
    },
    (err) => {
      console.error(err);
    }
  );

  const addTask = async () => {
    try {
      await addDoc(collection(db, "tasks"), {
        ...task,
        createdAt: Date.now(),
      });
      Toastify({
        text: "New Task created",
      }).showToast();
    } catch (error) {
      console.error(error);
    }
  };

  const deleteTask = async (id) => {
    try {
      await deleteDoc(doc(db, "tasks", id));
    } catch (error) {
      console.error(error);
    }
  };

  const editTask = (currenTask) => {
    currentId = currenTask.id;
    task.title = currenTask.title;
    task.description = currenTask.description;

    editStatus = true;
  };

  const updateTask = async () => {
    try {
      await updateDoc(doc(db, "tasks", currentId), task);
      Toastify({
        theme: "sunset",
        type: "success",
        timeout: 3000,
        text: "Task updated",
      }).showToast();
    } catch (error) {
      console.error(error);
    }
  };

  const handleSubmit = () => {
    if (!task.title) return;

    if (!editStatus) {
      addTask();
    } else {
      updateTask();
      editStatus = false;
      currentId = "";
    }

    task = { title: "", description: "" };
    inputElement.focus();
  };

  const onCancel = () => {
    editStatus = false;
    currentId = "";
    task = { title: "", description: "" };
  };

  onDestroy(unsub);
</script>


        