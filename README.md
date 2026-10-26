#PROG_ASSIGNMENT_SAHRJEREMIAHSOGBOR_905005761
PROGRAM SchoolTrackSL

// ---------- CONSTANTS ----------
CONSTANT PASS_MARK = 50
CONSTANT MIN_ATTENDANCE_RATE = 75
CONSTANT MAX_CONSECUTIVE_ABSENCES = 3
CONSTANT MAX_LOGIN_ATTEMPTS = 3

// ---------- MAIN MODULE ----------
BEGIN
  IF Login() = TRUE THEN
    SET running = TRUE
    WHILE running = TRUE DO
      DISPLAY "1 Attendance  2 Results  3 Reports  4 Exit"
      INPUT choice
      IF choice = 1 THEN
        CALL RecordAttendance()
      ELSE IF choice = 2 THEN
        CALL EnterResults()
      ELSE IF choice = 3 THEN
        CALL ViewReports()
      ELSE IF choice = 4 THEN
        SET running = FALSE
      ELSE
        DISPLAY "Invalid option"
      END IF
    END WHILE
  END IF
END

// ---------- MODULE 1: LOGIN ----------
FUNCTION Login() RETURNS Boolean
  SET attempts = 0
  WHILE attempts < MAX_LOGIN_ATTEMPTS DO
    INPUT username, password
    IF CheckCredentials(username, password) = TRUE THEN
      RETURN TRUE
    ELSE
      SET attempts = attempts + 1
      DISPLAY "Wrong username or password"
    END IF
  END WHILE
  DISPLAY "Account locked. Contact the head teacher."
  RETURN FALSE
END FUNCTION

// ---------- MODULE 2: ATTENDANCE ----------
PROCEDURE RecordAttendance()
  INPUT className, date
  SET studentList = LoadStudents(className)
  SET atRiskList = EMPTY LIST
  FOR EACH student IN studentList DO
    INPUT status                       // "P" or "A"
    SET student.totalDays = student.totalDays + 1
    IF status = "P" THEN
      SET student.presentDays = student.presentDays + 1
      SET student.streak = 0
    ELSE
      SET student.streak = student.streak + 1
    END IF
    SET rate = CalculateRate(student.presentDays, student.totalDays)
    SET isAtRisk = (rate < MIN_ATTENDANCE_RATE) OR
                   (student.streak >= MAX_CONSECUTIVE_ABSENCES)
    IF isAtRisk = TRUE THEN
      ADD student TO atRiskList
    END IF
    CALL SaveAttendance(student, date, status)
  END FOR
  DISPLAY class summary and atRiskList
END PROCEDURE

FUNCTION CalculateRate(presentDays, totalDays) RETURNS Real
  IF totalDays = 0 THEN
    RETURN 0
  END IF
  RETURN (presentDays / totalDays) * 100
END FUNCTION

// ---------- MODULE 3: RESULTS ----------
PROCEDURE EnterResults()
  INPUT studentID
  IF StudentExists(studentID) = FALSE THEN
    DISPLAY "Student not found"
    RETURN
  END IF
  SET total = 0
  SET count = 0
  FOR EACH subject IN subjectList DO
    REPEAT
      INPUT mark
      IF mark < 0 OR mark > 100 THEN
        DISPLAY "Invalid mark. Enter 0 to 100"
      END IF
    UNTIL mark >= 0 AND mark <= 100
    SET total = total + mark
    SET count = count + 1
  END FOR
  SET average = total / count
  SET grade = AssignGrade(average)
  CALL SaveResults(studentID, total, average, grade)
  DISPLAY report card for studentID
END PROCEDURE

FUNCTION AssignGrade(average) RETURNS Character
  IF average >= 80 THEN
    RETURN "A"
  ELSE IF average >= 70 THEN
    RETURN "B"
  ELSE IF average >= 60 THEN
    RETURN "C"
  ELSE IF average >= PASS_MARK THEN
    RETURN "D"
  ELSE
    RETURN "F"
  END IF
END FUNCTION

// ---------- MODULE 4: REPORTS ----------
PROCEDURE ViewReports()
  DISPLAY "1 Student report  2 Class attendance  3 At-risk list"
  INPUT option
  IF option = 1 THEN
    INPUT studentID
    CALL ShowStudentReport(studentID)
  ELSE IF option = 2 THEN
    INPUT className
    CALL ShowClassAttendance(className)
  ELSE IF option = 3 THEN
    CALL ShowAtRiskStudents()
  ELSE
    DISPLAY "Invalid option"
  END IF
END PROCEDURE

END PROGRAM
