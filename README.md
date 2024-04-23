# goit-rdb-fp
1.
-- Створення схеми pandemic
CREATE SCHEMA IF NOT EXISTS pandemic;

-- Оберіть схему за замовчуванням
USE pandemic;
![p_1](https://github.com/NGanch/goit-rdb-fp/assets/86801593/8b901687-9151-46be-8c7e-7bea7468e4ab)

2.
-- Створення таблиці для унікальних сутностей
CREATE TABLE IF NOT EXISTS entities (
    entity_id INT AUTO_INCREMENT PRIMARY KEY,
    entity_name VARCHAR(255) NOT NULL
);

-- Створення таблиці для унікальних кодів
CREATE TABLE IF NOT EXISTS codes (
    code_id INT AUTO_INCREMENT PRIMARY KEY,
    code VARCHAR(255) NOT NULL
);

-- Створення нормалізованої таблиці для кількості захворювань
CREATE TABLE IF NOT EXISTS normalized_infectious_cases (
    case_id INT AUTO_INCREMENT PRIMARY KEY,
    entity_id INT,
    code_id INT,
    year INT,
    number_rabies INT,
    FOREIGN KEY (entity_id) REFERENCES entities(entity_id),
    FOREIGN KEY (code_id) REFERENCES codes(code_id)
);

-- Вставка унікальних значень Entity в таблицу entities
INSERT INTO entities (entity_name)
SELECT DISTINCT Entity FROM infectious_cases;

-- Вставка унікальних значень Code в таблицу codes
INSERT INTO codes (code)
SELECT DISTINCT Code FROM infectious_cases;

-- Вставка нормалізованих даних у таблицю normalized_infectious_cases
INSERT INTO normalized_infectious_cases (entity_id, code_id, year, number_rabies)
SELECT e.entity_id, c.code_id, Year, Number_rabies
FROM infectious_cases i
JOIN entities e ON i.Entity = e.entity_name
JOIN codes c ON i.Code = c.code
WHERE Number_rabies <> ''; -- Фільтрація порожніх значень
![image](https://github.com/NGanch/goit-rdb-fp/assets/86801593/a6ca7fa3-2e98-4880-8eda-270db18c183b)
![image](https://github.com/NGanch/goit-rdb-fp/assets/86801593/67b0eccc-abce-4cc0-b1d8-6963c92b9bf4)
![image](https://github.com/NGanch/goit-rdb-fp/assets/86801593/94dbe5fb-f875-4996-8c12-4ca02f7c4e48)
![image](https://github.com/NGanch/goit-rdb-fp/assets/86801593/ad3074c6-3341-47d7-89e0-eb81af6fd0fc)

3.
-- Розрахунок середнього, мінімального, максимального значення та суми для атрибута Number_rabies
SELECT 
    entity_id,
    code_id,
    AVG(number_rabies) AS average_rabies,
    MIN(number_rabies) AS min_rabies,
    MAX(number_rabies) AS max_rabies,
    SUM(number_rabies) AS sum_rabies
FROM normalized_infectious_cases
WHERE number_rabies <> ''  -- Фільтрація порожніх значень
GROUP BY entity_id, code_id
ORDER BY average_rabies DESC
LIMIT 10;  -- Вибірка тільки 10 рядків
![image](https://github.com/NGanch/goit-rdb-fp/assets/86801593/7be9c37d-d25e-4a99-af06-5401fcda3ffa)

4.
-- Додавання нової колонки з датою першого січня відповідного року
ALTER TABLE normalized_infectious_cases ADD COLUMN start_of_year DATE;

UPDATE normalized_infectious_cases 
SET start_of_year = CONCAT(year, '-01-01') 
WHERE case_id > 0; -- припустимо, що case_id - це ключовий стовпець
![image](https://github.com/NGanch/goit-rdb-fp/assets/86801593/98925808-2098-4e07-9b33-f9f0456ab4fc)

-- Додавання нової колонки з поточною датою
ALTER TABLE normalized_infectious_cases ADD COLUMN current_date DATE;

UPDATE normalized_infectious_cases 
SET current_of_date = CURDATE() 
WHERE case_id > 0; -- припустимо, що case_id - це ключовий стовпець
![image](https://github.com/NGanch/goit-rdb-fp/assets/86801593/bc2232ad-4682-4ec2-8df8-ce5000277c2d)

-- Додавання нової колонки з різницею в роках
ALTER TABLE normalized_infectious_cases ADD COLUMN year_difference INT;

UPDATE normalized_infectious_cases 
SET year_difference = TIMESTAMPDIFF(YEAR, start_of_year, current_date) 
WHERE case_id > 0; -- припустимо, що case_id - це ключовий стовпець
![image](https://github.com/NGanch/goit-rdb-fp/assets/86801593/abc26726-93ff-4058-aaa1-d578b61b9eea)
![image](https://github.com/NGanch/goit-rdb-fp/assets/86801593/267c9441-ffed-48ad-bd4f-856b770200ae)

5.
DELIMITER $$
CREATE FUNCTION CalculateYearDifference (input_year INT)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE current_year INT;
    SET current_year = YEAR(CURDATE());
    RETURN current_year - input_year;
END $$
DELIMITER ;

-- Приклад використання функції для року 1996
SELECT CalculateYearDifference(1996) AS year_difference_from_1996;
![image](https://github.com/NGanch/goit-rdb-fp/assets/86801593/00863222-a131-4660-ace6-50ee2ed808e3)
