# REAL-TIME-VOLCANIC-ERRUPTION-SIMULATION
Postgresql 
# CREATE SCHEME
CREATE SCHEMA volcano_lab;
SET search_path TO volcano_lab;
# TERRAIN GRID (3D SURFACE)
CREATE TABLE terrain_grid (
    x INT,
    y INT,
    height FLOAT,
    temperature FLOAT DEFAULT 20,
    pressure FLOAT DEFAULT 1,
    PRIMARY KEY(x, y)
);
# INITIALIZE VOLCANO TERRAIN
CREATE OR REPLACE FUNCTION generate_terrain(size INT)
RETURNS VOID AS $$
DECLARE
    i INT;
    j INT;
    center FLOAT := size / 2;
BEGIN
    FOR i IN 1..size LOOP
        FOR j IN 1..size LOOP
            INSERT INTO terrain_grid(x, y, height)
            VALUES (
                i,
                j,
                100 * exp(-((i-center)^2 + (j-center)^2) / 200.0)
            );
        END LOOP;
    END LOOP;
END;
$$ LANGUAGE plpgsql;
# MAGMA PREESSURE BUILD-UP
CREATE OR REPLACE FUNCTION increase_pressure(rate FLOAT)
RETURNS VOID AS $$
BEGIN
    UPDATE terrain_grid
    SET pressure = pressure + (height / 100) * rate;
END;
$$ LANGUAGE plpgsql;
# ERRUPTION TRIGGER LOGIC
CREATE OR REPLACE FUNCTION check_eruption()
RETURNS TEXT AS $$
DECLARE
    max_pressure FLOAT;
BEGIN
    SELECT MAX(pressure) INTO max_pressure FROM terrain_grid;

    IF max_pressure > 5 THEN
        PERFORM erupt();
        RETURN '🔥 ERUPTION TRIGGERED!';
    ELSE
        RETURN 'Pressure Stable';
    END IF;
END;
$$ LANGUAGE plpgsql;
# ERRUPTION SIMULATOR
CREATE OR REPLACE FUNCTION erupt()
RETURNS VOID AS $$
DECLARE
    center_x INT;
    center_y INT;
BEGIN
    SELECT AVG(x), AVG(y)
    INTO center_x, center_y
    FROM terrain_grid;

    UPDATE terrain_grid
    SET 
        temperature = 1000,
        height = height - RANDOM() * 20
    WHERE 
        sqrt((x-center_x)^2 + (y-center_y)^2) < 5;

    -- Shockwave spread
    UPDATE terrain_grid
    SET temperature = temperature + 200 * exp(-sqrt((x-center_x)^2 + (y-center_y)^2)/10);
END;
$$ LANGUAGE plpgsql;
# LAVA COOLING
CREATE OR REPLACE FUNCTION cool_lava()
RETURNS VOID AS $$
BEGIN
    UPDATE terrain_grid
    SET temperature = temperature - 50
    WHERE temperature > 20;

    UPDATE terrain_grid
    SET height = height + 2
    WHERE temperature < 200 AND temperature > 20;
END;
$$ LANGUAGE plpgsql;
# SEISMIC WAVE GENERATOR
CREATE OR REPLACE FUNCTION seismic_wave(step INT)
RETURNS FLOAT AS $$
BEGIN
    IF step <= 0 THEN
        RETURN 0;
    ELSE
        RETURN sin(step) + seismic_wave(step - 1);
    END IF;
END;
$$ LANGUAGE plpgsql;
# ANALYTICAL HEAT MAP
CREATE VIEW heat_map AS
SELECT
    AVG(temperature) AS avg_temperature,
    MAX(temperature) AS max_temperature,
    AVG(pressure) AS avg_pressure
FROM terrain_grid;
# FULL SIMULATION RUN
SELECT increase_pressure(2);
SELECT check_eruption();
SELECT cool_lava();
SELECT * FROM heat_map;
# OUTPUT

