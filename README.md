# Assignment-1-exercise-2
import java.time.LocalTime;
import java.time.format.DateTimeParseException;
import java.util.*;
import java.util.Scanner;

enum Priority {
    LOW, MEDIUM, HIGH
}

// Task class represents a task
class Task {
    private String description;
    private LocalTime startTime;
    private LocalTime endTime;
    private Priority priority;
    private boolean completed;

    public Task(String description, LocalTime startTime, LocalTime endTime, Priority priority) {
        this.description = description;
        this.startTime = startTime;
        this.endTime = endTime;
        this.priority = priority;
        this.completed = false;
    }

    public LocalTime getStartTime() {
        return startTime;
    }

    public LocalTime getEndTime() {
        return endTime;
    }

    public Priority getPriority() {
        return priority;
    }

    public void markCompleted() {
        this.completed = true;
    }

    public boolean isCompleted() {
        return completed;
    }

    @Override
    public String toString() {
        return startTime + " - " + endTime + ": " + description + " [" + priority + "] " + (completed ? "(Completed)" : "");
    }
}

// TaskFactory to create Task objects
class TaskFactory {
    public static Task createTask(String description, String startTime, String endTime, String priority) {
        try {
            LocalTime start = LocalTime.parse(startTime);
            LocalTime end = LocalTime.parse(endTime);
            Priority taskPriority = Priority.valueOf(priority.toUpperCase());
            if(start.isAfter(end)) {
                throw new IllegalArgumentException("Start time cannot be after end time.");
            }
            return new Task(description, start, end, taskPriority);
        } catch (DateTimeParseException | IllegalArgumentException e) {
            throw new IllegalArgumentException("Invalid input: " + e.getMessage());
        }
    }
}

// Singleton ScheduleManager class
class ScheduleManager {
    private static ScheduleManager instance;
    private List<Task> tasks;
    private List<Observer> observers;

    private ScheduleManager() {
        tasks = new ArrayList<>();
        observers = new ArrayList<>();
    }

    public static ScheduleManager getInstance() {
        if (instance == null) {
            instance = new ScheduleManager();
        }
        return instance;
    }

    public void addObserver(Observer observer) {
        observers.add(observer);
    }

    private void notifyObservers(String message) {
        for (Observer observer : observers) {
            observer.update(message);
        }
    }

    public void addTask(Task task) {
        if (isTaskConflicting(task)) {
            notifyObservers("Task conflicts with an existing task.");
        } else {
            tasks.add(task);
            tasks.sort(Comparator.comparing(Task::getStartTime));
            Logger.log("Task added: " + task);
        }
    }

    public void removeTask(String description) {
        Optional<Task> taskToRemove = tasks.stream()
            .filter(task -> task.toString().contains(description))
            .findFirst();
        if (taskToRemove.isPresent()) {
            tasks.remove(taskToRemove.get());
            Logger.log("Task removed: " + description);
        } else {
            Logger.log("Task not found: " + description);
        }
    }

    public void viewTasks() {
        if (tasks.isEmpty()) {
            System.out.println("No tasks scheduled for the day.");
        } else {
            tasks.forEach(System.out::println);
        }
    }

    public void viewTasksByPriority(Priority priority) {
        tasks.stream()
            .filter(task -> task.getPriority() == priority)
            .forEach(System.out::println);
    }

    private boolean isTaskConflicting(Task newTask) {
        for (Task task : tasks) {
            if (!(newTask.getEndTime().isBefore(task.getStartTime()) || newTask.getStartTime().isAfter(task.getEndTime()))) {
                return true;
            }
        }
        return false;
    }
}

// Observer interface
interface Observer {
    void update(String message);
}

// ConflictObserver class implements the Observer interface
class ConflictObserver implements Observer {
    @Override
    public void update(String message) {
        System.out.println("Conflict Detected: " + message);
    }
}

// Logger class
class Logger {
    public static void log(String message) {
        System.out.println("[LOG] " + message);
    }
}

// Main application class
public class AstronautSchedulerApp {

    public static void main(String[] args) {
        ScheduleManager scheduleManager = ScheduleManager.getInstance();
        ConflictObserver conflictObserver = new ConflictObserver();
        scheduleManager.addObserver(conflictObserver);

        Scanner scanner = new Scanner(System.in);
        String input;
        boolean running = true;

        while (running) {
            System.out.println("\nAstronaut Daily Schedule Organizer");
            System.out.println("1. Add Task");
            System.out.println("2. Remove Task");
            System.out.println("3. View Tasks");
            System.out.println("4. View Tasks by Priority");
            System.out.println("5. Exit");
            System.out.print("Select an option: ");
            input = scanner.nextLine();

            switch (input) {
                case "1": // Add Task
                    System.out.print("Enter Task Description: ");
                    String description = scanner.nextLine();
                    System.out.print("Enter Start Time (HH:MM): ");
                    String startTime = scanner.nextLine();
                    System.out.print("Enter End Time (HH:MM): ");
                    String endTime = scanner.nextLine();
                    System.out.print("Enter Priority (LOW, MEDIUM, HIGH): ");
                    String priority = scanner.nextLine();

                    try {
                        Task task = TaskFactory.createTask(description, startTime, endTime, priority);
                        scheduleManager.addTask(task);
                        System.out.println("Task added successfully. No conflicts.");
                    } catch (IllegalArgumentException e) {
                        System.out.println("Error: " + e.getMessage());
                    }
                    break;

                case "2": // Remove Task
                    System.out.print("Enter Task Description to Remove: ");
                    String removeDescription = scanner.nextLine();
                    scheduleManager.removeTask(removeDescription);
                    break;

                case "3": // View All Tasks
                    scheduleManager.viewTasks();
                    break;

                case "4": // View Tasks by Priority
                    System.out.print("Enter Priority to View (LOW, MEDIUM, HIGH): ");
                    String priorityLevel = scanner.nextLine();
                    try {
                        Priority selectedPriority = Priority.valueOf(priorityLevel.toUpperCase());
                        scheduleManager.viewTasksByPriority(selectedPriority);
                    } catch (IllegalArgumentException e) {
                        System.out.println("Error: Invalid priority level.");
                    }
                    break;

                case "5": // Exit
                    running = false;
                    System.out.println("Exiting the application.");
                    break;

                default:
                    System.out.println("Invalid option. Please try again.");
                    break;
            }
        }
        scanner.close();
    }
}
