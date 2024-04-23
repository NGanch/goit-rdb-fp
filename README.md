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
    id INT AUTO_INCREMENT PRIMARY KEY,
    code VARCHAR(255) NOT NULL,
    name VARCHAR(255) NOT NULL
);

INSERT INTO entities (code, name)
SELECT DISTINCT code, entity FROM infectious_cases;

ALTER TABLE infectious_cases
ADD COLUMN entity_id INT,
ADD FOREIGN KEY (entity_id) REFERENCES entities(id);

UPDATE infectious_cases ic
JOIN entities e ON ic.Entity = e.name AND ic.Code = e.code
SET ic.entity_id = e.id
WHERE ic.id IN (SELECT id FROM infectious_cases WHERE entity_id IS NULL);

![image](https://github.com/NGanch/goit-rdb-fp/assets/86801593/a30b2e59-fd62-4963-8c09-7a70dfa75bd5)



3.
-- Розрахунок середнього, мінімального, максимального значення та суми для атрибута Number_rabies
  
SELECT entity_id, AVG(Number_rabies) AS avg_rabies,
       MIN(Number_rabies) AS min_rabies, MAX(Number_rabies) AS max_rabies,
       SUM(Number_rabies) AS total_rabies
FROM infectious_cases
WHERE Number_rabies IS NOT NULL
GROUP BY entity_id
ORDER BY avg_rabies DESC
LIMIT 10;
![image](https://github.com/NGanch/goit-rdb-fp/assets/86801593/ee0b8f3b-5f71-4864-bb9e-f1f5cb3d699e)


4.
SELECT Year,
       CONCAT(Year, '-01-01') AS start_of_year,
       CURDATE() AS current_date_column,
       TIMESTAMPDIFF(YEAR, CONCAT(Year, '-01-01'), CURDATE()) AS year_difference
FROM infectious_cases;

![image](https://github.com/NGanch/goit-rdb-fp/assets/86801593/0058c749-c95a-47fa-b09f-02787057fdc2)


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
