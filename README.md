# Теория баз данных.

## Тема: Триггеры.

#



1. Создать триггер, который позволяет только увеличивать размер фонда финансирования факультета.

CREATE TRIGGER validate_salary 
BEFORE UPDATE 
ON Teachers 
FOR EACH ROW 
IF OLD.Salary>NEW.Salary THEN SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = "The faculty's funding fund can only increase";
END IF


![текст](https://sun9-57.userapi.com/impg/8Gagvv1rgCxAJNC1Z_CjbUoldUBX7S2jVzGt8Q/yaApXdAvVhs.jpg?size=1920x1080&quality=95&sign=84518b063ebb755bef5429130757c78b&type=album) 

Если поставить заработную плату ниже предыдушей, то будет появляться предупреждение о недопустимости данного действия: 

![текст](https://sun9-50.userapi.com/impg/uR_yCbrHNXBUgqvVtA7eChMzH1TssOy9m_TXkg/8hpohTLbQbY.jpg?size=1920x1080&quality=95&sign=b3f6af7463fd1813b16f388892569dc2&type=album)

2. Создать триггер, который фиксирует в журнале сведений при добавлении нового преподавателя.

CREATE TRIGGER after_insert_teacher
AFTER INSERT
ON Teachers
OR EACH ROW BEGIN
INSERT INTO TeacherManipulations (`Date`, ActionId, TeacherId) VALUES (CURDATE(), 1, NEW.Id);
INSERT INTO TeacherAddedInfos (EmploymentDate,
Name, Salary, Surname, ManipulationId) SELECT EmploymentDate, Name, Salary, Surname, LAST_INSERT_ID() FROM Teachers WHERE Id = New.Id;
END


![текст](https://sun9-23.userapi.com/impg/5Rr40_RB_JJCG-drcbiBPsk7NrBXJP9nhPJMIg/A66ncN1EHf0.jpg?size=1920x1080&quality=95&sign=142241d11cb13aba8f4f806c8b829323&type=album)

3. Создать триггер, который фиксирует в журнале сведений при изменении данных о преподавателе. 

CREATE TRIGGER after_update_teacher
AFTER UPDATE
ON Teachers
FOR EACH ROW
BEGIN
DECLARE the_last_inserted_id INT;
INSERT INTO TeacherManipulations (`Date`, ActionId, TeacherId) VALUES (CURDATE(), 3, OLD.Id);
SET the_last_inserted_id = LAST_INSERT_ID();
INSERT INTO TeacherAddedInfos (EmploymentDate, Name, Surname, Salary, ManipulationId) VALUES (NEW.EmploymentDate, NEW.Name, NEW.Surname, NEW.Salary, the_last_inserted_id);
INSERT INTO TeacherDeletedInfos (EmploymentDate, Name, Surname, Salary, ManipulationId) VALUES (OLD.EmploymentDate, OLD.Name, OLD.Surname, OLD.Salary, the_last_inserted_id);
END

![текст](https://sun9-8.userapi.com/impg/tuX2KaMK-dasUVoyK17jXGrDdw_MjpBueJUt8A/BWxM517p-uc.jpg?size=1920x1080&quality=95&sign=d92d789410df451a24050bbfcb19a1c3&type=album)

4. Создать триггер, который фиксирует в журнале сведений при удалении преподавателя. 

CREATE TRIGGER after_delete_teacher
AFTER DELETE
ON Teachers
FOR EACH ROW
BEGIN
INSERT INTO TeacherManipulations (`Date`, ActionId, TeacherId) VALUES (CURDATE(), 2, OLD.Id);
INSERT INTO TeacherDeletedInfos (EmploymentDate, Name, Surname, Salary, ManipulationId) VALUES (OLD.EmploymentDate, OLD.Name, OLD.Surname, OLD.Salary, LAST_INSERT_ID());
END

![текст](https://sun9-49.userapi.com/impg/PUy_aFRaJdqhanvU2VXhFP_FyqsBtOZqqE_Czw/lztvAvhTBlY.jpg?size=1920x1080&quality=95&sign=f02805c05bcb6118628c5195d66b4528&type=album)