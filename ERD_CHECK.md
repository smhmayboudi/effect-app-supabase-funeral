# ERD Design Specification - Enhanced CHECK Constraints

1. ENHANCED CHECK CONSTRAINTS FOR EXISTING TABLES

### USERS Table - Enhanced Checks

```sql
-- ============ ADD TO EXISTING USERS TABLE ============
ALTER TABLE users
ADD CONSTRAINT valid_email_format 
    CHECK (email ~* '^[A-Za-z0-9._+%-]+@[A-Za-z0-9.-]+[.][A-Za-z]+$'),
ADD CONSTRAINT valid_phone_format 
    CHECK (phone IS NULL OR phone ~ '^\+?[1-9]\d{1,14}$'),
ADD CONSTRAINT valid_full_name 
    CHECK (full_name ~ '^[A-Za-zÀ-ÿ\s\.\-\'']{2,100}$'),
ADD CONSTRAINT role_transition_valid 
    CHECK (
        (role = 'customer' AND email_verified = TRUE) OR
        (role IN ('provider_admin', 'provider_staff') AND email_verified = TRUE) OR
        role = 'super_admin'
    ),
ADD CONSTRAINT last_login_sensible 
    CHECK (last_login_at IS NULL OR last_login_at <= NOW() + INTERVAL '5 minutes'),
ADD CONSTRAINT created_updated_order 
    CHECK (created_at <= updated_at OR updated_at IS NULL);
```

### ORGANIZATIONS Table - Enhanced Checks

```sql
-- ============ ADD TO EXISTING ORGANIZATIONS TABLE ============
ALTER TABLE organizations
ADD CONSTRAINT valid_email_format_org 
    CHECK (email ~* '^[A-Za-z0-9._+%-]+@[A-Za-z0-9.-]+[.][A-Za-z]+$'),
ADD CONSTRAINT valid_phone_org 
    CHECK (phone ~ '^\+?[1-9]\d{1,14}$'),
ADD CONSTRAINT valid_website_url 
    CHECK (
        website IS NULL OR 
        website ~ '^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$'
    ),
ADD CONSTRAINT valid_tax_id_format 
    CHECK (
        tax_id IS NULL OR 
        tax_id ~ '^[A-Za-z0-9\-]{8,20}$'
    ),
ADD CONSTRAINT valid_license_number 
    CHECK (
        license_number IS NULL OR 
        license_number ~ '^[A-Za-z0-9\-/]{5,50}$'
    ),
ADD CONSTRAINT rating_bounds 
    CHECK (rating >= 0 AND rating <= 5),
ADD CONSTRAINT total_reviews_non_negative 
    CHECK (total_reviews >= 0),
ADD CONSTRAINT status_transition_valid 
    CHECK (
        (status = 'pending' AND verification_status IN ('unverified', 'pending')) OR
        (status = 'active' AND verification_status = 'verified') OR
        (status IN ('suspended', 'inactive'))
    ),
ADD CONSTRAINT valid_coordinates 
    CHECK (
        (address->'coordinates'->>'lat')::FLOAT BETWEEN -90 AND 90 AND
        (address->'coordinates'->>'lng')::FLOAT BETWEEN -180 AND 180
    ),
ADD CONSTRAINT valid_address_structure 
    CHECK (
        address ? 'street' AND 
        address ? 'city' AND 
        address ? 'country' AND
        address ? 'coordinates' AND
        address->'coordinates' ? 'lat' AND
        address->'coordinates' ? 'lng'
    );
```

### ORGANIZATION_USERS Table - Enhanced Checks

```sql
-- ============ ADD TO EXISTING ORGANIZATION_USERS TABLE ============
ALTER TABLE organization_users
ADD CONSTRAINT valid_department 
    CHECK (
        department IS NULL OR 
        department IN ('management', 'sales', 'operations', 'support', 'finance')
    ),
ADD CONSTRAINT single_primary_contact 
    UNIQUE (organization_id, is_primary_contact) 
    WHERE is_primary_contact = TRUE,
ADD CONSTRAINT role_permissions_consistency 
    CHECK (
        (role = 'admin' AND permissions ? 'all') OR
        (role = 'manager' AND permissions ? 'manage_services' AND permissions ? 'view_financials') OR
        (role = 'staff' AND permissions ? 'manage_reservations') OR
        (role = 'viewer' AND NOT (permissions ? 'write'))
    ),
ADD CONSTRAINT created_before_updated 
    CHECK (created_at <= updated_at OR updated_at IS NULL);
```

### SERVICES Table - Enhanced Checks

```sql
-- ============ ADD TO EXISTING SERVICES TABLE ============
ALTER TABLE services
ADD CONSTRAINT valid_service_name 
    CHECK (name ~ '^[A-Za-zÀ-ÿ0-9\s\-\.\&]{3,100}$'),
ADD CONSTRAINT positive_duration 
    CHECK (duration_minutes IS NULL OR duration_minutes > 0),
ADD CONSTRAINT valid_sort_order 
    CHECK (sort_order >= 0),
ADD CONSTRAINT service_metadata_structure 
    CHECK (
        metadata ? 'requirements' AND
        metadata ? 'restrictions' AND
        metadata ? 'notes'
    ),
ADD CONSTRAINT service_type_specific_checks 
    CHECK (
        (service_type = 'transportation' AND duration_minutes IS NOT NULL) OR
        (service_type = 'burial' AND metadata->>'requires_plot' IS NOT NULL) OR
        service_type NOT IN ('transportation', 'burial')
    );
```

### LOCATIONS Table - Enhanced Checks

```sql
-- ============ ADD TO EXISTING LOCATIONS TABLE ============
ALTER TABLE locations
ADD CONSTRAINT valid_location_name 
    CHECK (name ~ '^[A-Za-zÀ-ÿ0-9\s\-\.\''\&\(\)]{3,100}$'),
ADD CONSTRAINT positive_capacity 
    CHECK (capacity IS NULL OR capacity > 0),
ADD CONSTRAINT valid_operating_hours 
    CHECK (
        operating_hours ? 'monday' AND
        operating_hours ? 'tuesday' AND
        operating_hours ? 'wednesday' AND
        operating_hours ? 'thursday' AND
        operating_hours ? 'friday' AND
        operating_hours ? 'saturday' AND
        operating_hours ? 'sunday'
    ),
ADD CONSTRAINT valid_operating_time_format 
    CHECK (
        operating_hours->'monday' ? 'open' AND
        operating_hours->'monday' ? 'close' AND
        operating_hours->'monday'->>'open' ~ '^[0-2][0-9]:[0-5][0-9]$' AND
        operating_hours->'monday'->>'close' ~ '^[0-2][0-9]:[0-5][0-9]$' AND
        operating_hours->'monday'->>'open' < operating_hours->'monday'->>'close'
    ),
ADD CONSTRAINT valid_images_array 
    CHECK (
        jsonb_array_length(images) <= 20 AND
        jsonb_array_length(images) >= 0
    ),
ADD CONSTRAINT facilities_array_valid 
    CHECK (
        jsonb_array_length(facilities) <= 50 AND
        jsonb_array_length(facilities) >= 0
    ),
ADD CONSTRAINT valid_coordinates_location 
    CHECK (
        (address->'coordinates'->>'lat')::FLOAT BETWEEN -90 AND 90 AND
        (address->'coordinates'->>'lng')::FLOAT BETWEEN -180 AND 180
    ),
ADD CONSTRAINT location_type_specific 
    CHECK (
        (location_type = 'cemetery' AND capacity IS NOT NULL) OR
        (location_type = 'crematorium' AND features ? 'cremation_chambers') OR
        location_type NOT IN ('cemetery', 'crematorium')
    );
```

### PRICING_PLANS Table - Enhanced Checks

```sql
-- ============ ADD TO EXISTING PRICING_PLANS TABLE ============
ALTER TABLE pricing_plans
ADD CONSTRAINT valid_plan_name 
    CHECK (name ~ '^[A-Za-zÀ-ÿ0-9\s\-\.]{3,50}$'),
ADD CONSTRAINT valid_currency 
    CHECK (currency IN ('USD', 'EUR', 'GBP', 'CAD', 'AUD')),
ADD CONSTRAINT valid_tax_rate 
    CHECK (tax_rate >= 0 AND tax_rate <= 100),
ADD CONSTRAINT positive_duration_days 
    CHECK (duration_days IS NULL OR duration_days > 0),
ADD CONSTRAINT features_array_limit 
    CHECK (jsonb_array_length(features) <= 100),
ADD CONSTRAINT valid_conditions 
    CHECK (
        conditions ? 'min_people' AND
        conditions ? 'max_people' AND
        conditions ? 'advance_notice_days'
    ),
ADD CONSTRAINT valid_people_range 
    CHECK (
        (conditions->>'min_people')::INTEGER >= 1 AND
        (conditions->>'max_people')::INTEGER >= (conditions->>'min_people')::INTEGER
    ),
ADD CONSTRAINT valid_advance_notice 
    CHECK ((conditions->>'advance_notice_days')::INTEGER >= 0),
ADD CONSTRAINT price_type_specific 
    CHECK (
        (price_type = 'hourly' AND duration_minutes IS NULL) OR
        (price_type = 'fixed' AND base_price > 0) OR
        (price_type = 'per_person' AND conditions ? 'base_people') OR
        price_type = 'custom'
    );
```

### AVAILABILITIES Table - Enhanced Checks

```sql
-- ============ ADD TO EXISTING AVAILABILITIES TABLE ============
ALTER TABLE availabilities
ADD CONSTRAINT valid_date_range 
    CHECK (date BETWEEN CURRENT_DATE AND CURRENT_DATE + INTERVAL '1 year'),
ADD CONSTRAINT valid_time_range 
    CHECK (
        start_time >= '00:00:00'::TIME AND
        end_time <= '23:59:59'::TIME AND
        start_time < end_time
    ),
ADD CONSTRAINT slot_duration_limit 
    CHECK (EXTRACT(EPOCH FROM (end_time - start_time)) / 60 >= 30),
ADD CONSTRAINT valid_capacity_limits 
    CHECK (
        max_capacity IS NULL OR 
        (max_capacity >= 1 AND max_capacity <= 1000)
    ),
ADD CONSTRAINT current_bookings_limit 
    CHECK (
        current_bookings >= 0 AND
        (max_capacity IS NULL OR current_bookings <= max_capacity)
    ),
ADD CONSTRAINT valid_price_multiplier 
    CHECK (price_multiplier >= 0.5 AND price_multiplier <= 5.0),
ADD CONSTRAINT no_overlapping_slots 
    EXCLUDE USING gist (
        organization_id WITH =,
        location_id WITH =,
        service_id WITH =,
        date WITH =,
        tsrange(
            (date + start_time)::timestamp,
            (date + end_time)::timestamp,
            '[)'
        ) WITH &&
    ),
ADD CONSTRAINT valid_slot_combinations 
    CHECK (
        (location_id IS NOT NULL AND service_id IS NULL) OR
        (location_id IS NULL AND service_id IS NOT NULL) OR
        (location_id IS NOT NULL AND service_id IS NOT NULL)
    );
```

### RESERVATIONS Table - Enhanced Checks

```sql
-- ============ ADD TO EXISTING RESERVATIONS TABLE ============
ALTER TABLE reservations
ADD CONSTRAINT valid_reference_number 
    CHECK (reference_number ~ '^RSV-\d{8}-[A-Z0-9]{9}$'),
ADD CONSTRAINT valid_scheduled_date 
    CHECK (
        scheduled_date IS NULL OR 
        scheduled_date BETWEEN CURRENT_DATE - INTERVAL '30 days' AND CURRENT_DATE + INTERVAL '1 year'
    ),
ADD CONSTRAINT valid_scheduled_time 
    CHECK (
        scheduled_time IS NULL OR
        (scheduled_time >= '00:00:00'::TIME AND scheduled_time <= '23:59:59'::TIME)
    ),
ADD CONSTRAINT deceased_person_valid 
    CHECK (
        deceased_person ? 'full_name' AND
        deceased_person ? 'date_of_birth' AND
        deceased_person ? 'date_of_death' AND
        deceased_person ? 'gender' AND
        (deceased_person->>'date_of_birth')::DATE < (deceased_person->>'date_of_death')::DATE AND
        (deceased_person->>'date_of_death')::DATE <= CURRENT_DATE AND
        deceased_person->>'gender' IN ('male', 'female', 'other')
    ),
ADD CONSTRAINT ceremony_details_valid 
    CHECK (
        ceremony_details ? 'type' AND
        ceremony_details ? 'date_time' AND
        ceremony_details ? 'duration_minutes' AND
        (ceremony_details->>'duration_minutes')::INTEGER > 0 AND
        (ceremony_details->>'duration_minutes')::INTEGER <= 480
    ),
ADD CONSTRAINT amounts_consistency 
    CHECK (
        subtotal + tax_amount = total_amount AND
        subtotal >= 0 AND
        tax_amount >= 0 AND
        total_amount >= 0
    ),
ADD CONSTRAINT valid_currency_reservation 
    CHECK (currency IN ('USD', 'EUR', 'GBP', 'CAD', 'AUD')),
ADD CONSTRAINT status_transition_consistency 
    CHECK (
        (status = 'draft' AND cancellation_date IS NULL AND cancellation_reason IS NULL) OR
        (status IN ('pending', 'confirmed', 'completed') AND cancellation_date IS NULL) OR
        (status IN ('cancelled', 'refunded') AND cancellation_date IS NOT NULL AND cancellation_reason IS NOT NULL)
    ),
ADD CONSTRAINT expiry_logic 
    CHECK (
        (status = 'draft' AND expires_at IS NOT NULL AND expires_at > created_at) OR
        (status != 'draft' AND expires_at IS NULL)
    ),
ADD CONSTRAINT scheduled_completion_consistency 
    CHECK (
        (status = 'completed' AND scheduled_date IS NOT NULL AND scheduled_date <= CURRENT_DATE) OR
        status != 'completed'
    );
```

### RESERVATION_SERVICES Table - Enhanced Checks

```sql
-- ============ ADD TO EXISTING RESERVATION_SERVICES TABLE ============
ALTER TABLE reservation_services
ADD CONSTRAINT valid_quantity 
    CHECK (quantity BETWEEN 1 AND 100),
ADD CONSTRAINT price_consistency 
    CHECK (unit_price * quantity = total_price),
ADD CONSTRAINT unit_price_positive 
    CHECK (unit_price >= 0),
ADD CONSTRAINT total_price_positive 
    CHECK (total_price >= 0),
ADD CONSTRAINT service_plan_consistency 
    CHECK (
        (service_id IS NOT NULL AND pricing_plan_id IS NULL) OR
        (service_id IS NOT NULL AND pricing_plan_id IS NOT NULL)
    );
```

### PAYMENTS Table - Enhanced Checks

```sql
-- ============ ADD TO EXISTING PAYMENTS TABLE ============
ALTER TABLE payments
ADD CONSTRAINT valid_payment_provider 
    CHECK (
        payment_provider IS NULL OR 
        payment_provider IN ('stripe', 'paypal', 'square', 'braintree')
    ),
ADD CONSTRAINT valid_provider_transaction_id 
    CHECK (
        provider_transaction_id IS NULL OR 
        provider_transaction_id ~ '^[a-z0-9_\-]{8,100}$'
    ),
ADD CONSTRAINT amount_positive 
    CHECK (amount > 0),
ADD CONSTRAINT valid_currency_payment 
    CHECK (currency IN ('USD', 'EUR', 'GBP', 'CAD', 'AUD')),
ADD CONSTRAINT status_consistency 
    CHECK (
        (status IN ('pending', 'processing') AND completed_at IS NULL) OR
        (status IN ('completed', 'failed', 'refunded', 'cancelled') AND completed_at IS NOT NULL)
    ),
ADD CONSTRAINT receipt_url_format 
    CHECK (
        receipt_url IS NULL OR 
        receipt_url ~ '^https?:\/\/[^\s]+$'
    ),
ADD CONSTRAINT metadata_structure 
    CHECK (
        metadata ? 'card_last4' AND
        metadata ? 'card_brand' AND
        metadata ? 'billing_address'
    ),
ADD CONSTRAINT created_before_completed 
    CHECK (created_at <= completed_at OR completed_at IS NULL);
```

### DOCUMENTS Table - Enhanced Checks

```sql
-- ============ ADD TO EXISTING DOCUMENTS TABLE ============
ALTER TABLE documents
ADD CONSTRAINT valid_file_name 
    CHECK (file_name ~ '^[A-Za-z0-9_\-\s\.]{1,200}\.(pdf|docx|png|jpg|jpeg)$'),
ADD CONSTRAINT valid_file_size 
    CHECK (file_size > 0 AND file_size <= 10485760), -- 10MB max
ADD CONSTRAINT valid_mime_type 
    CHECK (
        mime_type IN (
            'application/pdf',
            'application/msword',
            'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
            'image/png',
            'image/jpeg',
            'image/jpg'
        )
    ),
ADD CONSTRAINT signed_consistency 
    CHECK (
        (is_signed = TRUE AND signed_at IS NOT NULL) OR
        (is_signed = FALSE AND signed_at IS NULL)
    ),
ADD CONSTRAINT signed_after_created 
    CHECK (signed_at >= created_at OR signed_at IS NULL);
```

### REVIEWS Table - Enhanced Checks

```sql
-- ============ ADD TO EXISTING REVIEWS TABLE ============
ALTER TABLE reviews
ADD CONSTRAINT rating_scale 
    CHECK (rating BETWEEN 1 AND 5),
ADD CONSTRAINT valid_title_length 
    CHECK (LENGTH(title) BETWEEN 5 AND 200),
ADD CONSTRAINT valid_comment_length 
    CHECK (LENGTH(comment) BETWEEN 10 AND 2000),
ADD CONSTRAINT response_consistency 
    CHECK (
        (response IS NOT NULL AND responded_at IS NOT NULL) OR
        (response IS NULL AND responded_at IS NULL)
    ),
ADD CONSTRAINT responded_after_created 
    CHECK (responded_at >= created_at OR responded_at IS NULL),
ADD CONSTRAINT verified_review_consistency 
    CHECK (
        (is_verified = TRUE AND reservation_id IS NOT NULL) OR
        is_verified = FALSE
    );
```

### NOTIFICATIONS Table - Enhanced Checks

```sql
-- ============ ADD TO EXISTING NOTIFICATIONS TABLE ============
ALTER TABLE notifications
ADD CONSTRAINT valid_title_length_notif 
    CHECK (LENGTH(title) BETWEEN 5 AND 200),
ADD CONSTRAINT valid_message_length 
    CHECK (LENGTH(message) BETWEEN 10 AND 1000),
ADD CONSTRAINT read_consistency 
    CHECK (
        (is_read = TRUE AND read_at IS NOT NULL) OR
        (is_read = FALSE AND read_at IS NULL)
    ),
ADD CONSTRAINT read_after_created 
    CHECK (read_at >= created_at OR read_at IS NULL),
ADD CONSTRAINT valid_sent_via_array 
    CHECK (
        jsonb_array_length(sent_via) >= 1 AND
        jsonb_array_length(sent_via) <= 3
    ),
ADD CONSTRAINT valid_channels 
    CHECK (
        sent_via <@ '["in_app", "email", "sms", "push"]'::jsonb
    );
```

2. NEW GLOBAL CONSTRAINTS AND DOMAINS

```sql
-- ============ ADD THESE GLOBAL DOMAINS ============

-- Create custom domains for reusable constraints
CREATE DOMAIN email_domain AS VARCHAR(255)
    CHECK (VALUE ~* '^[A-Za-z0-9._+%-]+@[A-Za-z0-9.-]+[.][A-Za-z]+$');

CREATE DOMAIN phone_domain AS VARCHAR(20)
    CHECK (VALUE ~ '^\+?[1-9]\d{1,14}$');

CREATE DOMAIN currency_code AS VARCHAR(3)
    CHECK (VALUE IN ('USD', 'EUR', 'GBP', 'CAD', 'AUD', 'JPY', 'CHF', 'CNY'));

CREATE DOMAIN percentage AS DECIMAL(5,2)
    CHECK (VALUE >= 0 AND VALUE <= 100);

CREATE DOMAIN rating_value AS DECIMAL(3,2)
    CHECK (VALUE >= 0 AND VALUE <= 5);

CREATE DOMAIN positive_money AS DECIMAL(10,2)
    CHECK (VALUE >= 0);

CREATE DOMAIN url_domain AS TEXT
    CHECK (VALUE ~ '^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$');

-- Apply domains to existing tables
ALTER TABLE users 
    ALTER COLUMN email TYPE email_domain,
    ALTER COLUMN phone TYPE phone_domain;

ALTER TABLE organizations
    ALTER COLUMN email TYPE email_domain,
    ALTER COLUMN phone TYPE phone_domain,
    ALTER COLUMN rating TYPE rating_value,
    ALTER COLUMN website TYPE url_domain;

ALTER TABLE pricing_plans
    ALTER COLUMN tax_rate TYPE percentage,
    ALTER COLUMN currency TYPE currency_code;

ALTER TABLE reservations
    ALTER COLUMN currency TYPE currency_code;

ALTER TABLE payments
    ALTER COLUMN currency TYPE currency_code;
```

3. CROSS-TABLE CONSISTENCY CONSTRAINTS

```sql
-- ============ ADD THESE CROSS-TABLE CONSTRAINTS ============

-- 1. Reservation must have at least one service
CREATE OR REPLACE FUNCTION check_reservation_has_services()
RETURNS TRIGGER AS $$
BEGIN
    IF NOT EXISTS (
        SELECT 1 FROM reservation_services 
        WHERE reservation_id = NEW.id
    ) THEN
        RAISE EXCEPTION 'Reservation must have at least one service';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE CONSTRAINT TRIGGER reservation_services_check
    AFTER INSERT ON reservations
    DEFERRABLE INITIALLY DEFERRED
    FOR EACH ROW
    EXECUTE FUNCTION check_reservation_has_services();

-- 2. Completed reservation must have payment
CREATE OR REPLACE FUNCTION check_completed_reservation_payment()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.status = 'completed' AND NOT EXISTS (
        SELECT 1 FROM payments 
        WHERE reservation_id = NEW.id 
        AND status = 'completed'
    ) THEN
        RAISE EXCEPTION 'Completed reservation must have a successful payment';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER completed_reservation_payment_check
    BEFORE UPDATE OF status ON reservations
    FOR EACH ROW
    WHEN (NEW.status = 'completed')
    EXECUTE FUNCTION check_completed_reservation_payment();

-- 3. Service must belong to same organization as reservation
CREATE OR REPLACE FUNCTION check_service_organization_consistency()
RETURNS TRIGGER AS $$
DECLARE
    service_org_id UUID;
    reservation_org_id UUID;
BEGIN
    SELECT organization_id INTO service_org_id 
    FROM services WHERE id = NEW.service_id;
    
    SELECT organization_id INTO reservation_org_id 
    FROM reservations WHERE id = NEW.reservation_id;
    
    IF service_org_id != reservation_org_id THEN
        RAISE EXCEPTION 'Service must belong to the same organization as reservation';
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER service_organization_consistency_check
    BEFORE INSERT OR UPDATE ON reservation_services
    FOR EACH ROW
    EXECUTE FUNCTION check_service_organization_consistency();

-- 4. Pricing plan must belong to same organization as service
CREATE OR REPLACE FUNCTION check_pricing_plan_organization()
RETURNS TRIGGER AS $$
DECLARE
    service_org_id UUID;
    plan_org_id UUID;
BEGIN
    IF NEW.pricing_plan_id IS NOT NULL THEN
        SELECT organization_id INTO service_org_id 
        FROM services WHERE id = NEW.service_id;
        
        SELECT organization_id INTO plan_org_id 
        FROM pricing_plans WHERE id = NEW.pricing_plan_id;
        
        IF service_org_id != plan_org_id THEN
            RAISE EXCEPTION 'Pricing plan must belong to the same organization as service';
        END IF;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER pricing_plan_organization_check
    BEFORE INSERT OR UPDATE ON reservation_services
    FOR EACH ROW
    EXECUTE FUNCTION check_pricing_plan_organization();
```

4. TEMPORAL CONSTRAINTS

```sql
-- ============ ADD THESE TEMPORAL CONSTRAINTS ============

-- 1. Cannot book in the past
CREATE OR REPLACE FUNCTION prevent_past_booking()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.scheduled_date < CURRENT_DATE OR
       (NEW.scheduled_date = CURRENT_DATE AND NEW.scheduled_time < CURRENT_TIME) THEN
        RAISE EXCEPTION 'Cannot book services in the past';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER prevent_past_booking_trigger
    BEFORE INSERT OR UPDATE ON reservations
    FOR EACH ROW
    EXECUTE FUNCTION prevent_past_booking();

-- 2. Minimum advance notice for bookings (based on service)
CREATE OR REPLACE FUNCTION check_advance_notice()
RETURNS TRIGGER AS $$
DECLARE
    min_notice_hours INTEGER;
    booking_datetime TIMESTAMP;
BEGIN
    -- Get minimum notice from service metadata
    SELECT COALESCE(
        (metadata->>'min_notice_hours')::INTEGER, 
        24
    ) INTO min_notice_hours
    FROM services s
    JOIN reservation_services rs ON rs.service_id = s.id
    WHERE rs.reservation_id = NEW.id
    LIMIT 1;
    
    booking_datetime := (NEW.scheduled_date + NEW.scheduled_time);
    
    IF booking_datetime < (NOW() + (min_notice_hours || ' hours')::INTERVAL) THEN
        RAISE EXCEPTION 'Booking must be made at least % hours in advance', min_notice_hours;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER check_advance_notice_trigger
    BEFORE INSERT OR UPDATE ON reservations
    FOR EACH ROW
    WHEN (NEW.scheduled_date IS NOT NULL AND NEW.scheduled_time IS NOT NULL)
    EXECUTE FUNCTION check_advance_notice();

-- 3. Cannot modify completed reservations
CREATE OR REPLACE FUNCTION prevent_modify_completed_reservation()
RETURNS TRIGGER AS $$
BEGIN
    IF OLD.status = 'completed' AND (
        NEW.status != OLD.status OR
        NEW.deceased_person != OLD.deceased_person OR
        NEW.scheduled_date != OLD.scheduled_date OR
        NEW.scheduled_time != OLD.scheduled_time
    ) THEN
        RAISE EXCEPTION 'Cannot modify completed reservations';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER prevent_modify_completed_reservation_trigger
    BEFORE UPDATE ON reservations
    FOR EACH ROW
    EXECUTE FUNCTION prevent_modify_completed_reservation();
```

5. BUSINESS RULE CONSTRAINTS

```sql
-- ============ ADD THESE BUSINESS RULE CONSTRAINTS ============

-- 1. Maximum services per reservation
CREATE OR REPLACE FUNCTION check_max_services_per_reservation()
RETURNS TRIGGER AS $$
DECLARE
    service_count INTEGER;
BEGIN
    SELECT COUNT(*) INTO service_count
    FROM reservation_services
    WHERE reservation_id = NEW.reservation_id;
    
    IF service_count > 10 THEN
        RAISE EXCEPTION 'Maximum 10 services per reservation';
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER check_max_services_trigger
    BEFORE INSERT ON reservation_services
    FOR EACH ROW
    EXECUTE FUNCTION check_max_services_per_reservation();

-- 2. Cannot book overlapping time slots at same location
CREATE OR REPLACE FUNCTION check_overlapping_reservations()
RETURNS TRIGGER AS $$
BEGIN
    IF EXISTS (
        SELECT 1 FROM reservations r
        WHERE r.location_id = NEW.location_id
        AND r.scheduled_date = NEW.scheduled_date
        AND r.status IN ('confirmed', 'pending')
        AND r.id != NEW.id
        AND tsrange(
            (r.scheduled_date + r.scheduled_time)::timestamp,
            (r.scheduled_date + r.scheduled_time + 
             INTERVAL '1 minute' * (r.ceremony_details->>'duration_minutes')::INTEGER
            )::timestamp,
            '[)'
        ) && tsrange(
            (NEW.scheduled_date + NEW.scheduled_time)::timestamp,
            (NEW.scheduled_date + NEW.scheduled_time + 
             INTERVAL '1 minute' * (NEW.ceremony_details->>'duration_minutes')::INTEGER
            )::timestamp,
            '[)'
        )
    ) THEN
        RAISE EXCEPTION 'Overlapping reservation at this location and time';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER check_overlapping_reservations_trigger
    BEFORE INSERT OR UPDATE ON reservations
    FOR EACH ROW
    WHEN (NEW.location_id IS NOT NULL AND NEW.scheduled_date IS NOT NULL AND NEW.scheduled_time IS NOT NULL)
    EXECUTE FUNCTION check_overlapping_reservations();

-- 3. Minimum time between death and service (based on local regulations)
CREATE OR REPLACE FUNCTION check_minimum_time_after_death()
RETURNS TRIGGER AS $$
DECLARE
    death_date DATE;
    service_date DATE;
    min_days INTEGER;
BEGIN
    death_date := (NEW.deceased_person->>'date_of_death')::DATE;
    service_date := NEW.scheduled_date;
    min_days := 1; -- Default minimum 1 day
    
    -- Could be extended based on location/religion regulations
    
    IF service_date < death_date + min_days THEN
        RAISE EXCEPTION 'Service must be at least % days after death', min_days;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER check_minimum_time_after_death_trigger
    BEFORE INSERT OR UPDATE ON reservations
    FOR EACH ROW
    WHEN (NEW.scheduled_date IS NOT NULL)
    EXECUTE FUNCTION check_minimum_time_after_death();
```

6. AUDIT CONSTRAINTS

```sql
-- ============ ADD THESE AUDIT CONSTRAINTS ============

-- 1. Ensure created_at is never in the future
CREATE OR REPLACE FUNCTION validate_timestamps()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.created_at > NOW() + INTERVAL '5 minutes' THEN
        NEW.created_at := NOW();
    END IF;
    
    IF NEW.updated_at > NOW() + INTERVAL '5 minutes' THEN
        NEW.updated_at := NOW();
    END IF;
    
    IF NEW.updated_at < NEW.created_at THEN
        NEW.updated_at := NEW.created_at;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Apply to all tables with created_at/updated_at
CREATE TRIGGER validate_users_timestamps
    BEFORE INSERT OR UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION validate_timestamps();

CREATE TRIGGER validate_organizations_timestamps
    BEFORE INSERT OR UPDATE ON organizations
    FOR EACH ROW
    EXECUTE FUNCTION validate_timestamps();

-- Repeat for all other tables...
```

7. SUMMARY OF ADDED CHECK CONSTRAINTS:

### Data Validation (Format & Structure):

- Email format validation for users & organizations
- Phone number format (E.164 standard)
- URL validation for websites
- File name and type validation
- JSON structure validation for addresses, metadata
- Coordinate range validation (-90 to 90, -180 to 180)

### Business Logic:

- Status transition consistency
- Date/time logical constraints
- Financial amount consistency
- Capacity and booking limits
- Minimum advance notice periods
- No overlapping reservations
- Service-organization consistency

### Temporal Constraints:

- Cannot book in the past
- Minimum time after death
- Created/updated timestamp order
- Expiry logic for draft reservations
- Completion date constraints

### Cross-Table Integrity:

- Reservation must have services
- Completed reservation must have payment
- Service belongs to same organization
- Pricing plan organization consistency

### Domain-Specific Rules:

- Funeral service specific rules
- Religious/cultural considerations
- Legal compliance (minimum days after death)
- Maximum services per reservation

These constraints ensure data integrity at the database level, preventing invalid data from being inserted, maintaining business rule consistency, and reducing application-level validation complexity.
