# MongoDB task

## About

This is MongoDB task to Design database for Zen class programme. <br />
Access the [mongoplayground](https://mongoplayground.net/p/6Uy-E58t93T) to resolve the [Queries](#queries--back-to-contents).


## Table of contents

1. [Database Design Screenshots](#database-design-screenshots--back-to-contents)
2. [Queries](#queries--back-to-contents)

## Database Design Screenshots | [Back to contents](#table-of-contents)

### Zenclass programme database

![Zenclass programme database](dbScreenshots/zen_class_programme.png?raw=true "Zenclass programme database")

### Users Collection

![Users Collection](dbScreenshots/zen_class_programme.users.png?raw=true "Users Collection")

### Codekata Collection

![Codekata Collection](dbScreenshots/zen_class_programme.codekata.png?raw=true "Codekata Collection")

### Attendance Collection

![Attendance Collection](dbScreenshots/zen_class_programme.attendance.png?raw=true "Attendance Collection")

### Topics Collection

![Topics Collection](dbScreenshots/zen_class_programme.topics.png?raw=true "Topics Collection")

### Tasks Collection

![Tasks Collection](dbScreenshots/zen_class_programme.tasks.png?raw=true "Tasks Collection")

### Company drives Collection

![Company_drives Collection](dbScreenshots/zen_class_programme.company_drives.png?raw=true "Company drives Collection")

### Mentors Collection

![Mentors Collection](dbScreenshots/zen_class_programme.company_drives.png?raw=true "Mentors Collection")

## Queries | [Back to contents](#table-of-contents)

### To switch databases or create database

- `Query`

```
use <database>
```

### Find all the topics and tasks which are thought in the month of October

- `Query`

```
db.topics.aggregate([
  {
    $lookup: {
      from: "tasks",
      localField: "_id",
      foreignField: "topic_id",
      as: "tasks_info",
    },
  },
  {
    $match: {
      $and: [
        { topics_date: { $gte: "2020-10-01", $lte: "2020-10-31" } },
        {
          "tasks_info.task_given_date": {
            $gte: "2020-10-01",
            $lte: "2020-10-31",
          },
        },
      ],
    },
  },
  {
    $addFields: {
      topic_info: { topic_name: "$topic_name", topics_date: "$topics_date" },
    },
  },
  { $project: { _id: 0, topic_name: 0, topics_date: 0 } },
]);
```

- `Sample Output` <br/><br/>
  ![sample output 1](outputScreenshots/sample_output_1.png?raw=true "Sample output 1")

### Find all the company drives which appeared between 15 oct-2020 and 31-oct-2020

- `Query`

```
db.company_drives.find({
  drive_date: { $gte: "2020-10-15", $lte: "2020-10-31" },
});
```

- `Sample Output` <br/><br/>
  ![sample output 2](outputScreenshots/sample_output_2.png?raw=true "Sample output 2")

### Find all the company drives and students who are appeared for the placement.

- `Query`

```
db.company_drives.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "appeared_students",
      foreignField: "_id",
      as: "students_appeared",
    },
  },
  { $project: { _id: 1, appeared_students: 0 } },
]);
```

- `Sample Output` <br/><br/>
  ![sample output 3](outputScreenshots/sample_output_3.png?raw=true "Sample output 3")

### Find the number of problems solved by the user in codekata.

- `Query`

```
db.codekata.aggregate([
  { $match: { user: ObjectId("664c5c58f25b6eb0dc46b799") } },
  {
    $group: { _id: "$user", total_problem_solved: { $sum: "$solved_problem" } },
  },
  {
    $lookup: {
      from: "users",
      localField: "_id",
      foreignField: "_id",
      as: "student",
    },
  },
  { $project: { _id: 0 } },
]);
```

- `Sample Output` <br/><br/>
  ![sample output 4](outputScreenshots/sample_output_4.png?raw=true "Sample output 4")

### Find all the mentors with who has the mentee's count more than 15.

- `Query`

```
db.mentors.find({ mentees_count: { $gt: 15 } });
```

- `Sample Output` <br/><br/>
  ![sample output 5](outputScreenshots/sample_output_5.png?raw=true "Sample output 5")

### Find the number of users who are absent and task is not submitted between 15 oct-2020 and 31-oct-2020.

- `Query`

```
db.attendance.aggregate([
  {
    $lookup: {
      from: "tasks",
      localField: "topic_id",
      foreignField: "topic_id",
      as: "task_info",
    },
  },
  {
    $match: {
      $and: [
        { attendance_date: { $gte: "2020-10-15", $lte: "2020-10-31" } },
        {
          "task_info.task_given_date": {
            $gte: "2020-10-15",
            $lte: "2020-10-31",
          },
        },
      ],
    },
  },
  {
    $project: { _id: 0, topic_id: 0, attendance_date: 0, students_present: 0 },
  },
  {
    $group: {
      _id: "null",
      not_submitted_students_arr: {
        $push: { $size: { $first: "$task_info.not_submitted_students" } },
      },
      absent_student_arr: { $push: { $size: "$students_absent" } },
    },
  },
  {
    $project: {
      _id: 0,
      number_of_absent_students: { $sum: "$absent_student_arr" },
      number_of_not_submitted_students: { $sum: "$not_submitted_students_arr" },
    },
  },
]);

```

- `Sample Output` <br/><br/>
  ![sample output 6](outputScreenshots/sample_output_6.png?raw=true "Sample output 6")