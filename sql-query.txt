USE ProjectD;
GO

-- --------------------------------------------
-- Table for user accounts
-- --------------------------------------------
CREATE TABLE user_accounts (
    user_id         INT            PRIMARY KEY IDENTITY(1,1),
    username        VARCHAR(50)    NOT NULL UNIQUE,
    email           VARCHAR(255)   NOT NULL UNIQUE,
    password_hash   VARCHAR(255)   NOT NULL,
    created_at      DATETIME       NOT NULL DEFAULT GETDATE(),
    updated_at      DATETIME       NOT NULL DEFAULT GETDATE()
);
GO

-- --------------------------------------------
-- Table for sign languages
-- --------------------------------------------
CREATE TABLE sign_languages (
    language_id     INT            PRIMARY KEY IDENTITY(1,1),
    code            VARCHAR(10)    NOT NULL UNIQUE,
    name            VARCHAR(100)   NOT NULL,
    country         VARCHAR(100),
    created_at      DATETIME       NOT NULL DEFAULT GETDATE(),
    updated_at      DATETIME       NOT NULL DEFAULT GETDATE()
);
GO

-- --------------------------------------------
-- Table for gestures
-- --------------------------------------------
CREATE TABLE gestures (
    gesture_id      INT            PRIMARY KEY IDENTITY(1,1),
    language_id     INT            NOT NULL FOREIGN KEY REFERENCES sign_languages(language_id),
    gesture_code    VARCHAR(255)   NOT NULL,
    name            VARCHAR(100),
    description     TEXT,
    created_at      DATETIME       NOT NULL DEFAULT GETDATE(),
    updated_at      DATETIME       NOT NULL DEFAULT GETDATE(),
    CONSTRAINT UQ_gesture_language UNIQUE(language_id, gesture_code)
);
GO

-- --------------------------------------------
-- Table for default translations
-- --------------------------------------------
CREATE TABLE gesture_translations (
    translation_id  INT            PRIMARY KEY IDENTITY(1,1),
    gesture_id      INT            NOT NULL FOREIGN KEY REFERENCES gestures(gesture_id),
    target_locale   VARCHAR(10)    NOT NULL,
    text_value      VARCHAR(255)   NOT NULL,
    created_at      DATETIME       NOT NULL DEFAULT GETDATE(),
    updated_at      DATETIME       NOT NULL DEFAULT GETDATE(),
    CONSTRAINT UQ_gesture_locale UNIQUE(gesture_id, target_locale)
);
GO

-- --------------------------------------------
-- Table for user custom translations
-- --------------------------------------------
CREATE TABLE user_custom_translations (
    custom_id       INT            PRIMARY KEY IDENTITY(1,1),
    user_id         INT            NOT NULL FOREIGN KEY REFERENCES user_accounts(user_id),
    gesture_id      INT            NOT NULL FOREIGN KEY REFERENCES gestures(gesture_id),
    custom_text     VARCHAR(255)   NOT NULL,
    created_at      DATETIME       NOT NULL DEFAULT GETDATE(),
    updated_at      DATETIME       NOT NULL DEFAULT GETDATE(),
    CONSTRAINT UQ_custom_user_gesture UNIQUE(user_id, gesture_id)
);
GO

-- --------------------------------------------
-- Table for gesture landmarks
-- --------------------------------------------
CREATE TABLE gesture_landmarks (
    landmark_id     INT            PRIMARY KEY IDENTITY(1,1),
    gesture_id      INT            NOT NULL FOREIGN KEY REFERENCES gestures(gesture_id),
    hand_label      VARCHAR(5)     NOT NULL,
    landmark_index  INT            NOT NULL,
    x_norm          FLOAT          NOT NULL,
    y_norm          FLOAT          NOT NULL,
    x_world         FLOAT,
    y_world         FLOAT,
    z_world         FLOAT,
    created_at      DATETIME       NOT NULL DEFAULT GETDATE()
);
GO

-- --------------------------------------------
-- Table for output sequences
-- --------------------------------------------
CREATE TABLE output_sequences (
    sequence_id     INT            PRIMARY KEY IDENTITY(1,1),
    language_id     INT            NOT NULL FOREIGN KEY REFERENCES sign_languages(language_id),
    input_text      TEXT           NOT NULL,
    description     TEXT,
    created_at      DATETIME       NOT NULL DEFAULT GETDATE()
);
GO

-- --------------------------------------------
-- Table for output sequence steps
-- --------------------------------------------
CREATE TABLE output_sequence_steps (
    step_id         INT            PRIMARY KEY IDENTITY(1,1),
    sequence_id     INT            NOT NULL FOREIGN KEY REFERENCES output_sequences(sequence_id),
    gesture_id      INT            NOT NULL FOREIGN KEY REFERENCES gestures(gesture_id),
    step_order      INT            NOT NULL,
    created_at      DATETIME       NOT NULL DEFAULT GETDATE()
);
GO

-- --------------------------------------------
-- Table for gesture predictions
-- --------------------------------------------
CREATE TABLE gesture_predictions (
    prediction_id   INT            PRIMARY KEY IDENTITY(1,1),
    user_id         INT            FOREIGN KEY REFERENCES user_accounts(user_id),
    timestamp       DATETIME       NOT NULL DEFAULT GETDATE(),
    input_data      TEXT,
    predicted_label VARCHAR(255),
    confidence      FLOAT,
    gesture_id      INT            FOREIGN KEY REFERENCES gestures(gesture_id)
);
GO

-- --------------------------------------------
-- Table for prediction feedback
-- --------------------------------------------
CREATE TABLE prediction_feedback (
    feedback_id     INT            PRIMARY KEY IDENTITY(1,1),
    prediction_id   INT            NOT NULL FOREIGN KEY REFERENCES gesture_predictions(prediction_id),
    is_correct      BIT            NOT NULL,
    user_comment    TEXT,
    submitted_at    DATETIME       NOT NULL DEFAULT GETDATE()
);
GO

-- --------------------------------------------
-- Indexes
-- --------------------------------------------
CREATE INDEX idx_gestures_language
    ON gestures(language_id);

CREATE INDEX idx_translations_gesture
    ON gesture_translations(gesture_id);

CREATE INDEX idx_custom_user_gesture
    ON user_custom_translations(user_id, gesture_id);

CREATE INDEX idx_landmarks_gesture
    ON gesture_landmarks(gesture_id);

CREATE INDEX idx_output_sequence_steps
    ON output_sequence_steps(sequence_id, step_order);
GO

-- --------------------------------------------
-- Optional: Insert Dutch Sign Language (DGS)
-- --------------------------------------------
INSERT INTO sign_languages (code, name, country)
VALUES ('DGS', 'Dutch Sign Language', 'Netherlands');
GO
