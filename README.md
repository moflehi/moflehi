CREATE TABLE users (
  user_id SERIAL PRIMARY KEY,
  first_name VARCHAR(50) NOT NULL,
  last_name VARCHAR(50) NOT NULL,
  email VARCHAR(100) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  phone VARCHAR(20),
  created_at TIMESTAMP DEFAULT NOW(),
  loyalty_points INT DEFAULT 0
);

CREATE TABLE airports (
  airport_code VARCHAR(3) PRIMARY KEY, -- e.g., JFK, LAX
  airport_name VARCHAR(100) NOT NULL,
  address TEXT NOT NULL,
  shuttle_frequency INT DEFAULT 15, -- minutes
  timezone VARCHAR(50) NOT NULL
);

CREATE TABLE parking_lots (
  lot_id SERIAL PRIMARY KEY,
  airport_code VARCHAR(3) REFERENCES airports(airport_code),
  lot_name VARCHAR(50) NOT NULL, -- e.g., "Lot A", "Covered Garage"
  lot_type VARCHAR(20) CHECK (lot_type IN ('covered', 'uncovered', 'valet')),
  base_daily_rate DECIMAL(6,2) NOT NULL,
  total_capacity INT NOT NULL,
  available_spots INT NOT NULL,
  is_active BOOLEAN DEFAULT TRUE,
  amenities TEXT[] -- Array of amenities: {shuttle, car_wash, ev_charging}
);

CREATE TABLE reservations (
  reservation_id SERIAL PRIMARY KEY,
  user_id INT REFERENCES users(user_id),
  lot_id INT REFERENCES parking_lots(lot_id),
  vehicle_license VARCHAR(20) NOT NULL,
  vehicle_make VARCHAR(50),
  vehicle_model VARCHAR(50),
  check_in TIMESTAMP NOT NULL,
  check_out TIMESTAMP NOT NULL,
  total_price DECIMAL(8,2) NOT NULL,
  status VARCHAR(20) DEFAULT 'pending' CHECK (status IN ('pending', 'confirmed', 'cancelled')),
  created_at TIMESTAMP DEFAULT NOW(),
  CONSTRAINT valid_duration CHECK (check_out > check_in)
);

CREATE TABLE payments (
  payment_id SERIAL PRIMARY KEY,
  reservation_id INT REFERENCES reservations(reservation_id),
  stripe_payment_id VARCHAR(255) NOT NULL,
  amount DECIMAL(8,2) NOT NULL,
  payment_method VARCHAR(20) CHECK (payment_method IN ('card', 'apple_pay', 'google_pay')),
  status VARCHAR(20) NOT NULL CHECK (status IN ('succeeded', 'failed', 'refunded')),
  processed_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE admin_rates (
  rate_id SERIAL PRIMARY KEY,
  lot_id INT REFERENCES parking_lots(lot_id),
  effective_date DATE NOT NULL,
  daily_rate DECIMAL(6,2) NOT NULL,
  updated_by INT REFERENCES users(user_id), -- Admin user
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE shuttle_schedules (
  shuttle_id SERIAL PRIMARY KEY,
  airport_code VARCHAR(3) REFERENCES airports(airport_code),
  departure_time TIME NOT NULL,
  terminal VARCHAR(50) NOT NULL,
  frequency INT DEFAULT 15 -- minutes
);

-- Indexes for faster queries
CREATE INDEX idx_reservations_dates ON reservations(check_in, check_out);
CREATE INDEX idx_parking_availability ON parking_lots(airport_code, lot_type, available_spots);
CREATE INDEX idx_user_reservations ON reservations(user_id);
