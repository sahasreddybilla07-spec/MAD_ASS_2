import 'package:flutter/material.dart';
import 'package:device_preview/device_preview.dart';

void main() {
runApp(DevicePreview(
enabled: true,
builder: (context) => TaskApp(),
));
}

class Task {
String title;
String category;
bool isDone;
DateTime dueDate;

Task({
required this.title,
required this.category,
this.isDone = false,
required this.dueDate,

});
}

class TaskApp extends StatelessWidget {
@override
Widget build(BuildContext context) {
return MaterialApp(
debugShowCheckedModeBanner: false,
home: HomeScreen(),
useInheritedMediaQuery: true,
locale: DevicePreview.locale(context),
builder: DevicePreview.appBuilder,
);
}
}

class HomeScreen extends StatefulWidget {
@override
_HomeScreenState createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {

int selectedIndex = 0;

List<Task> tasks = [
Task(title: "Submit project report", category: "Urgent", dueDate: DateTime.now()),
Task(title: "Review Flutter chapter 4 & 5", category: "Study", dueDate: DateTime.now()),
Task(title: "Attend team standup meeting", category: "Work", dueDate: DateTime.now()),
Task(title: "Buy groceries", category: "Personal", dueDate: DateTime.now()),
Task(title: "Call dentist", category: "Personal", isDone: true, dueDate: DateTime.now()),
Task(title: "Push code to GitHub", category: "Work", isDone: true, dueDate: DateTime.now()),
];

void addTask(Task task) {
setState(() {
tasks.add(task);
});
}

@override
Widget build(BuildContext context) {
List<Widget> screens = [
TaskScreen(tasks: tasks, addTask: addTask),
Center(child: Text("Today View (optional)")),

StatsScreen(tasks: tasks),
];

return Scaffold(
body: screens[selectedIndex],
bottomNavigationBar: BottomNavigationBar(
currentIndex: selectedIndex,
onTap: (i) => setState(() => selectedIndex = i),
items: [
BottomNavigationBarItem(icon: Icon(Icons.grid_view), label: "Tasks"),
BottomNavigationBarItem(icon: Icon(Icons.list), label: "Today"),
BottomNavigationBarItem(icon: Icon(Icons.access_time), label: "Stats"),
],
),
);
}
}

class TaskScreen extends StatefulWidget {
final List<Task> tasks;
final Function(Task) addTask;

TaskScreen({required this.tasks, required this.addTask});

@override
_TaskScreenState createState() => _TaskScreenState();
}

class _TaskScreenState extends State<TaskScreen> {
@override
Widget build(BuildContext context) {
int done = widget.tasks.where((t) => t.isDone).length;
double percent = widget.tasks.isEmpty ? 0 : done / widget.tasks.length;

return SafeArea(
child: Column(
children: [
Padding(
padding: EdgeInsets.all(16),
child: Column(
crossAxisAlignment: CrossAxisAlignment.start,
children: [
Text("My Tasks", style: TextStyle(fontSize: 28, fontWeight: FontWeight.bold)),
SizedBox(height: 5),

Text("${done} of ${widget.tasks.length} done"),
SizedBox(height: 10),
LinearProgressIndicator(value: percent),
],
),
),

Expanded(
child: ListView(
children: widget.tasks.map((task) {
return ListTile(
leading: Checkbox(
value: task.isDone,
onChanged: (val) {
setState(() {
task.isDone = val!;
});
},
),
title: Text(
task.title,
style: TextStyle(

decoration: task.isDone ? TextDecoration.lineThrough : null,
),
),
trailing: Chip(
label: Text(task.category),
),
);
}).toList(),
),
),

Padding(
padding: EdgeInsets.all(16),
child: ElevatedButton(
onPressed: () async {
Task? newTask = await Navigator.push(
context,
MaterialPageRoute(builder: (_) => AddTaskScreen()),
);
if (newTask != null) widget.addTask(newTask);
},
child: Text("Add Task"),

),
)
],
),
);
}
}

class AddTaskScreen extends StatefulWidget {
@override
_AddTaskScreenState createState() => _AddTaskScreenState();
}

class _AddTaskScreenState extends State<AddTaskScreen> {
TextEditingController controller = TextEditingController();
String category = "Work";

@override
Widget build(BuildContext context) {
return Scaffold(
appBar: AppBar(title: Text("New Task")),
body: Padding(

padding: EdgeInsets.all(16),
child: Column(
children: [
TextField(
controller: controller,
decoration: InputDecoration(labelText: "Task title"),
),
SizedBox(height: 20),

DropdownButton<String>(
value: category,
items: ["Work", "Personal", "Study", "Urgent"]
.map((e) => DropdownMenuItem(value: e, child: Text(e)))
.toList(),
onChanged: (val) {
setState(() {
category = val!;
});
},
),

SizedBox(height: 20),

ElevatedButton(
onPressed: () {
Navigator.pop(
context,
Task(
title: controller.text,
category: category,
dueDate: DateTime.now(),
),
);
},
child: Text("Save"),
)
],
),
),
);
}
}

class StatsScreen extends StatelessWidget {

final List<Task> tasks;

StatsScreen({required this.tasks});

@override
Widget build(BuildContext context) {
int total = tasks.length;
int done = tasks.where((t) => t.isDone).length;
int active = tasks.where((t) => !t.isDone).length;

return SafeArea(
child: Padding(
padding: EdgeInsets.all(16),
child: Column(
children: [
Text("Statistics", style: TextStyle(fontSize: 28, fontWeight: FontWeight.bold)),
SizedBox(height: 20),

Row(
mainAxisAlignment: MainAxisAlignment.spaceAround,
children: [
statBox("Total", total, Colors.blue),

statBox("Done", done, Colors.green),
],
),

SizedBox(height: 20),

Row(
mainAxisAlignment: MainAxisAlignment.spaceAround,
children: [
statBox("Active", active, Colors.orange),
statBox("Overdue", 1, Colors.red),
],
),

SizedBox(height: 40),

Text(
'Developed by Sahas Reddy Billa\nRoll No: 160124737193',
textAlign: TextAlign.center,
style: TextStyle(
fontSize: 12,
color: Colors.grey,

),
),
],
),
),
);
}

Widget statBox(String title, int value, Color color) {
return Container(
width: 140,
height: 100,
decoration: BoxDecoration(
color: Colors.grey.shade200,
borderRadius: BorderRadius.circular(12),
),
child: Column(
mainAxisAlignment: MainAxisAlignment.center,
children: [
Text("$value", style: TextStyle(fontSize: 26, color: color)),
Text(title),
],

),
);
}
}