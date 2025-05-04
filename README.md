# flutter-bloc-documentation
// Project structure:
// lib/
// ├── main.dart
// ├── models/
// │   └── student.dart
// ├── data/
// │   └── student_repository.dart
// ├── bloc/
// │   ├── student_bloc.dart
// │   ├── student_event.dart
// │   └── student_state.dart
// └── screens/
//     ├── home_screen.dart
//     ├── student_list_screen.dart
//     ├── add_student_screen.dart
//     └── edit_student_screen.dart

// 1. First, let's create the Student model
// lib/models/student.dart
import 'package:equatable/equatable.dart';

class Student extends Equatable {
  final String id;
  final String name;
  final int age;
  final String school;

  const Student({
    required this.id,
    required this.name,
    required this.age,
    required this.school,
  });

  Student copyWith({
    String? id,
    String? name,
    int? age,
    String? school,
  }) {
    return Student(
      id: id ?? this.id,
      name: name ?? this.name,
      age: age ?? this.age,
      school: school ?? this.school,
    );
  }

  @override
  List<Object?> get props => [id, name, age, school];

  Map<String, dynamic> toMap() {
    return {
      'id': id,
      'name': name,
      'age': age,
      'school': school,
    };
  }

  factory Student.fromMap(Map<String, dynamic> map) {
    return Student(
      id: map['id'] ?? '',
      name: map['name'] ?? '',
      age: map['age'] ?? 0,
      school: map['school'] ?? '',
    );
  }
}

// 2. Student Repository
// lib/data/student_repository.dart
import 'dart:async';
import 'package:uuid/uuid.dart';
import '../models/student.dart';

class StudentRepository {
  final List<Student> _students = [];
  final _uuid = Uuid();

  // Get all students
  List<Student> getAllStudents() => _students;

  // Get student by id
  Student? getStudentById(String id) {
    try {
      return _students.firstWhere((student) => student.id == id);
    } catch (e) {
      return null;
    }
  }

  // Add student
  Future<Student> addStudent(String name, int age, String school) async {
    final student = Student(
      id: _uuid.v4(),
      name: name,
      age: age,
      school: school,
    );
    _students.add(student);
    return student;
  }

  // Update student
  Future<Student?> updateStudent(Student student) async {
    final index = _students.indexWhere((s) => s.id == student.id);
    if (index != -1) {
      _students[index] = student;
      return student;
    }
    return null;
  }

  // Delete student
  Future<void> deleteStudent(String id) async {
    _students.removeWhere((student) => student.id == id);
  }
}

// 3. BLoC - Events
// lib/bloc/student_event.dart
import 'package:equatable/equatable.dart';
import '../models/student.dart';

abstract class StudentEvent extends Equatable {
  const StudentEvent();

  @override
  List<Object?> get props => [];
}

class LoadStudents extends StudentEvent {}

class AddStudentEvent extends StudentEvent {
  final String name;
  final int age;
  final String school;

  const AddStudentEvent({
    required this.name,
    required this.age,
    required this.school,
  });

  @override
  List<Object?> get props => [name, age, school];
}

class UpdateStudentEvent extends StudentEvent {
  final Student student;

  const UpdateStudentEvent(this.student);

  @override
  List<Object?> get props => [student];
}

class DeleteStudentEvent extends StudentEvent {
  final String id;

  const DeleteStudentEvent(this.id);

  @override
  List<Object?> get props => [id];
}

// 4. BLoC - States
// lib/bloc/student_state.dart
import 'package:equatable/equatable.dart';
import '../models/student.dart';

abstract class StudentState extends Equatable {
  const StudentState();

  @override
  List<Object?> get props => [];
}

class StudentInitial extends StudentState {}

class StudentLoading extends StudentState {}

class StudentsLoaded extends StudentState {
  final List<Student> students;

  const StudentsLoaded(this.students);

  @override
  List<Object?> get props => [students];
}

class StudentOperationSuccess extends StudentState {
  final String message;

  const StudentOperationSuccess(this.message);

  @override
  List<Object?> get props => [message];
}

class StudentOperationFailure extends StudentState {
  final String error;

  const StudentOperationFailure(this.error);

  @override
  List<Object?> get props => [error];
}

// 5. BLoC - Business Logic
// lib/bloc/student_bloc.dart
import 'package:flutter_bloc/flutter_bloc.dart';
import '../data/student_repository.dart';
import 'student_event.dart';
import 'student_state.dart';

class StudentBloc extends Bloc<StudentEvent, StudentState> {
  final StudentRepository _studentRepository;

  StudentBloc({required StudentRepository studentRepository})
      : _studentRepository = studentRepository,
        super(StudentInitial()) {
    on<LoadStudents>(_onLoadStudents);
    on<AddStudentEvent>(_onAddStudent);
    on<UpdateStudentEvent>(_onUpdateStudent);
    on<DeleteStudentEvent>(_onDeleteStudent);
  }

  void _onLoadStudents(LoadStudents event, Emitter<StudentState> emit) {
    emit(StudentLoading());
    try {
      final students = _studentRepository.getAllStudents();
      emit(StudentsLoaded(students));
    } catch (e) {
      emit(StudentOperationFailure('Failed to load students: ${e.toString()}'));
    }
  }

  Future<void> _onAddStudent(
      AddStudentEvent event, Emitter<StudentState> emit) async {
    emit(StudentLoading());
    try {
      await _studentRepository.addStudent(event.name, event.age, event.school);
      final students = _studentRepository.getAllStudents();
      emit(StudentsLoaded(students));
      emit(const StudentOperationSuccess('Student added successfully'));
    } catch (e) {
      emit(StudentOperationFailure('Failed to add student: ${e.toString()}'));
    }
  }

  Future<void> _onUpdateStudent(
      UpdateStudentEvent event, Emitter<StudentState> emit) async {
    emit(StudentLoading());
    try {
      await _studentRepository.updateStudent(event.student);
      final students = _studentRepository.getAllStudents();
      emit(StudentsLoaded(students));
      emit(const StudentOperationSuccess('Student updated successfully'));
    } catch (e) {
      emit(StudentOperationFailure('Failed to update student: ${e.toString()}'));
    }
  }

  Future<void> _onDeleteStudent(
      DeleteStudentEvent event, Emitter<StudentState> emit) async {
    emit(StudentLoading());
    try {
      await _studentRepository.deleteStudent(event.id);
      final students = _studentRepository.getAllStudents();
      emit(StudentsLoaded(students));
      emit(const StudentOperationSuccess('Student deleted successfully'));
    } catch (e) {
      emit(StudentOperationFailure('Failed to delete student: ${e.toString()}'));
    }
  }
}

// 6. Screens
// lib/screens/home_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../bloc/student_bloc.dart';
import '../bloc/student_event.dart';
import 'student_list_screen.dart';

class HomeScreen extends StatelessWidget {
  const HomeScreen({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Student Management System'),
      ),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            context.read<StudentBloc>().add(LoadStudents());
            Navigator.push(
              context,
              MaterialPageRoute(builder: (context) => const StudentListScreen()),
            );
          },
          child: const Text('Manage Students'),
        ),
      ),
    );
  }
}

// lib/screens/student_list_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../bloc/student_bloc.dart';
import '../bloc/student_event.dart';
import '../bloc/student_state.dart';
import '../models/student.dart';
import 'add_student_screen.dart';
import 'edit_student_screen.dart';

class StudentListScreen extends StatelessWidget {
  const StudentListScreen({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Student List'),
      ),
      body: BlocConsumer<StudentBloc, StudentState>(
        listener: (context, state) {
          if (state is StudentOperationSuccess) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(content: Text(state.message)),
            );
          } else if (state is StudentOperationFailure) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(content: Text(state.error), backgroundColor: Colors.red),
            );
          }
        },
        builder: (context, state) {
          if (state is StudentLoading) {
            return const Center(child: CircularProgressIndicator());
          } else if (state is StudentsLoaded) {
            return _buildStudentList(context, state.students);
          } else {
            return const Center(child: Text('Press the + button to add students'));
          }
        },
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          Navigator.push(
            context,
            MaterialPageRoute(builder: (context) => const AddStudentScreen()),
          );
        },
        child: const Icon(Icons.add),
      ),
    );
  }

  Widget _buildStudentList(BuildContext context, List<Student> students) {
    if (students.isEmpty) {
      return const Center(child: Text('No students found'));
    }

    return ListView.builder(
      itemCount: students.length,
      itemBuilder: (context, index) {
        final student = students[index];
        return ListTile(
          title: Text(student.name),
          subtitle: Text('Age: ${student.age} | School: ${student.school}'),
          trailing: Row(
            mainAxisSize: MainAxisSize.min,
            children: [
              IconButton(
                icon: const Icon(Icons.edit),
                onPressed: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(
                      builder: (context) => EditStudentScreen(student: student),
                    ),
                  );
                },
              ),
              IconButton(
                icon: const Icon(Icons.delete),
                onPressed: () {
                  _showDeleteConfirmationDialog(context, student);
                },
              ),
            ],
          ),
        );
      },
    );
  }

  void _showDeleteConfirmationDialog(BuildContext context, Student student) {
    showDialog(
      context: context,
      builder: (BuildContext context) {
        return AlertDialog(
          title: const Text('Delete Student'),
          content: Text('Are you sure you want to delete ${student.name}?'),
          actions: <Widget>[
            TextButton(
              child: const Text('Cancel'),
              onPressed: () {
                Navigator.of(context).pop();
              },
            ),
            TextButton(
              child: const Text('Delete'),
              onPressed: () {
                context.read<StudentBloc>().add(DeleteStudentEvent(student.id));
                Navigator.of(context).pop();
              },
            ),
          ],
        );
      },
    );
  }
}

// lib/screens/add_student_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../bloc/student_bloc.dart';
import '../bloc/student_event.dart';

class AddStudentScreen extends StatefulWidget {
  const AddStudentScreen({Key? key}) : super(key: key);

  @override
  State<AddStudentScreen> createState() => _AddStudentScreenState();
}

class _AddStudentScreenState extends State<AddStudentScreen> {
  final _formKey = GlobalKey<FormState>();
  final TextEditingController _nameController = TextEditingController();
  final TextEditingController _ageController = TextEditingController();
  final TextEditingController _schoolController = TextEditingController();

  @override
  void dispose() {
    _nameController.dispose();
    _ageController.dispose();
    _schoolController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Add Student'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Form(
          key: _formKey,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              TextFormField(
                controller: _nameController,
                decoration: const InputDecoration(
                  labelText: 'Name',
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter student name';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              TextFormField(
                controller: _ageController,
                decoration: const InputDecoration(
                  labelText: 'Age',
                  border: OutlineInputBorder(),
                ),
                keyboardType: TextInputType.number,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter student age';
                  }
                  if (int.tryParse(value) == null) {
                    return 'Please enter a valid number';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              TextFormField(
                controller: _schoolController,
                decoration: const InputDecoration(
                  labelText: 'School',
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter school name';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 24),
              ElevatedButton(
                onPressed: () {
                  if (_formKey.currentState!.validate()) {
                    context.read<StudentBloc>().add(
                          AddStudentEvent(
                            name: _nameController.text,
                            age: int.parse(_ageController.text),
                            school: _schoolController.text,
                          ),
                        );
                    Navigator.pop(context);
                  }
                },
                child: const Text('Add Student'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

// lib/screens/edit_student_screen.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../bloc/student_bloc.dart';
import '../bloc/student_event.dart';
import '../models/student.dart';

class EditStudentScreen extends StatefulWidget {
  final Student student;

  const EditStudentScreen({Key? key, required this.student}) : super(key: key);

  @override
  State<EditStudentScreen> createState() => _EditStudentScreenState();
}

class _EditStudentScreenState extends State<EditStudentScreen> {
  final _formKey = GlobalKey<FormState>();
  late TextEditingController _nameController;
  late TextEditingController _ageController;
  late TextEditingController _schoolController;

  @override
  void initState() {
    super.initState();
    _nameController = TextEditingController(text: widget.student.name);
    _ageController = TextEditingController(text: widget.student.age.toString());
    _schoolController = TextEditingController(text: widget.student.school);
  }

  @override
  void dispose() {
    _nameController.dispose();
    _ageController.dispose();
    _schoolController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Edit Student'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Form(
          key: _formKey,
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.stretch,
            children: [
              TextFormField(
                controller: _nameController,
                decoration: const InputDecoration(
                  labelText: 'Name',
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter student name';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              TextFormField(
                controller: _ageController,
                decoration: const InputDecoration(
                  labelText: 'Age',
                  border: OutlineInputBorder(),
                ),
                keyboardType: TextInputType.number,
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter student age';
                  }
                  if (int.tryParse(value) == null) {
                    return 'Please enter a valid number';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 16),
              TextFormField(
                controller: _schoolController,
                decoration: const InputDecoration(
                  labelText: 'School',
                  border: OutlineInputBorder(),
                ),
                validator: (value) {
                  if (value == null || value.isEmpty) {
                    return 'Please enter school name';
                  }
                  return null;
                },
              ),
              const SizedBox(height: 24),
              ElevatedButton(
                onPressed: () {
                  if (_formKey.currentState!.validate()) {
                    final updatedStudent = widget.student.copyWith(
                      name: _nameController.text,
                      age: int.parse(_ageController.text),
                      school: _schoolController.text,
                    );
                    context.read<StudentBloc>().add(
                          UpdateStudentEvent(updatedStudent),
                        );
                    Navigator.pop(context);
                  }
                },
                child: const Text('Update Student'),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

// 7. Main Application
// lib/main.dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'bloc/student_bloc.dart';
import 'data/student_repository.dart';
import 'screens/home_screen.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return RepositoryProvider(
      create: (context) => StudentRepository(),
      child: BlocProvider(
        create: (context) => StudentBloc(
          studentRepository: RepositoryProvider.of<StudentRepository>(context),
        ),
        child: MaterialApp(
          title: 'Student Management System',
          theme: ThemeData(
            primarySwatch: Colors.blue,
            visualDensity: VisualDensity.adaptivePlatformDensity,
          ),
          home: const HomeScreen(),
        ),
      ),
    );
  }
}
