# Functional Design

## Probem Statement

### Problem Domain

**Fitness Tracking**: I'm an active gym user who used to rely on simple note taking for tracking workouts but then switched to a workout tracking app
that proved to be very effective with tracking workouts as well as progress. I understand both the value of these apps and their limitations such as 
lack of actionable insights from collected progress data. This domain interests me because progress tracking is key to fitness success even though people 
struggle with it despite having these digital tools. 

### Problem

Fitness enthusiasts have a hard time maintaining consistent progress tracking and lack guidance on when and how to advance their workouts, which leads to 
training plateaus and abandoned fitness goals. 

### Stakeholders

1.**Gym User**

*Role*: primary user that is tracking their workouts and seeking progress.

*Impact of the problem*: experiences demotivation when unable to see clear progress patterns which may lead to workout inconsistencies.

2. **Gym Owner**

*Role*: business owner interested in member retention.

*Impact of the problem*: loses revenue when members become discouraged cancelling their memberships. 

3. **Equipment Manufacturer**
   
*Role*: companies whose products could have integrated tracking systems.

*Impact of the problem*: the problem creates market opportunity for manufacturers to differentiate through better tracking integration and user guidance
systems.

4. **Personal Trainer**

*Role*: professional who guides clients and monitors their development.

*Impact of the problem*: could actually benefit from clients' tracking struggles since poor self-tracking creates dependency on trainer expertise for
progression guidance and progress monitoring, potentially increasing session frequency and trainer value.

### Evidence and Comparables

**Evidence**

- [31% exercise intervention dropout rate in STRRIDE trials](https://pmc.ncbi.nlm.nih.gov/articles/PMC9165469/): A large-scale study of 947 participants
   found that nearly one-third dropped out of structured exercise programs, with 67% of dropouts occurring during the initial ramp-up phase.
   This demonstrates the common struggle with maintaining consistent fitness tracking and progression, especially when people can't see
   early progress or lack guidance on how to advance.

- [Moderate 63-68% adherence rates in unsupervised fitness programs](https://ijbnpa.biomedcentral.com/articles/10.1186/s12966-023-01535-w): Meta-analysis
   reveals that even motivated participants in fitness studies only complete about two-thirds of prescribed sessions in real-world settings. This highlights
   how the lack of guidance and progress visibility in unsupervised training leads to reduced adherence to programs.
   
- [Plateau effect occurs around 4-6 months into fitness routines](https://www1.villanova.edu/content/dam/villanova/studentlife/documents/healthpromotion/Hitting%20a%20Strength%20Plateau.pdf):
   Research identifies that strength progress typically levels off after 4-6 months of consistent training. This timing coincides with when many people
   abandon fitness goals, suggesting that the inability to navigate progression decisions contributes to workout abandonment.

- [50% of people stop exercise programs within first six months](https://www.sciencedirect.com/science/article/abs/pii/S1469029217307963): Multiple
   studies confirm that approximately half of people who begin exercise programs quit within six months, missing out on important health benefits.
   This aligns directly with the progression plateau timeline, suggesting that lack of visible progress and guidance on advancement contributes to program abandonment.

**Comparables**

- [Strong](https://www.strong.app/): Popular weightlifting app but with limited progression insights
- [MyFitnessPal](https://www.myfitnesspal.com/): Comprehensive but complex logging process, focuses more on nutrition than workout optimization
- [Strava](https://www.strava.com/): Great for cardio but limited strength training features and progression analysis

## Application Pitch

### Name

**RepRight**

### Motivation

RepRight eliminates workout tracking friction and provides intelligent progression guidance, helping fitness enthusiasts stay consistent and break through 
plateaus to achieve their strength goals.

### Key Features

- **Smart Set Logging** : Pre-filled workout data based on previous sessions - tap to confirm or adjust. Reduces logging from 10+ taps to 1-2 taps per set,
  eliminating friction that causes users to abandon tracking. Gym Users maintain consistency; Personal Trainers get complete data;
  Gym Owners see better retention.
- **Auto-Progression Recommendations** : Algorithm analyzes performance patterns and suggests specific next steps: "Try 135lbs for 5 reps" or
  "Add a fourth set." Eliminates guesswork that causes plateaus or injuries. Gym Users get personalized guidance without paying for trainers;
  Personal Trainers focus on technique over programming; Equipment Manufacturers see sustained product usage.
- **Plateau Detection & Intervention**: Detects stalled progress (same performance 3+ weeks) and suggests evidence-based fixes: deload weeks,
  volume adjustments, exercise variations. Addresses the critical 4-6 month dropout window where 50% quit. Gym Users break through plateaus;
  Gym Owners retain members past the initial few months.

## Concept Design 

### 1. WorkoutLog [User, Exercise]

    concept WorkoutLog [User, Exercise]

    purpose track workout performance over time

    principle 
      after logging sets with exercise, weight, and reps,
      users can retrieve their workout history to see past performance

    state
      a set of WorkoutSets with
        a user User
        an exercise Exercise
        a weight Number
        a reps Number
        a date Date

    actions
      logSet (user: User, exercise: Exercise, weight: Number, reps: Number)
        requires weight >= 0 and reps > 0
        effects create new WorkoutSet with current date

      getLastWorkout (user: User, exercise: Exercise): (weight: Number, reps: Number, date: Date)
        requires at least one workout exists for this user and exercise
        effects return most recent set for this exercise 

      getWorkoutHistory (user: User, exercise: Exercise, startDate: Date, endDate: Date): (sets: set of WorkoutSet)
        requires startDate is before end date 
        effects return all sets for this exercise within the date range

### 2. ProgressionGuidance [User, Exercise]

    concept ProgressionGuidance [User, Exercise]

    purpose provide intelligent recommendations for workout progression

    principle
      after analyzing workout history patterns,
      system suggests next workout parameters to optimize progress

    state
      a set of Recommendations with
        a user User
        an exercise Exercise
        a suggestedWeight Number
        a suggestedReps Number
        a reasoning String

    actions
      generateRecommendation (user: User, exercise: Exercise, recentSets: set of WorkoutSet): (recommendation: Recommendation)
        requires recentSets contains at least 3 sets from past 4 weeks
        effects analyze performance patterns and create recommendation
          if user completed all prescribed reps easily, suggest weight increase
          if user struggled with current weight, suggest maintaining or reducing

      getRecommendation (user: User, exercise: Exercise): (weight: Number, reps: Number, reasoning: String)
        requires recommendation exists for this user and exercise
        effects return most recent recommendation

      detectPlateau (user: User, exercise: Exercise, historicalSets: set of WorkoutSet): (isPlateaued: Boolean)
        requires historicalSets spans at least 3 weeks
        effects return true if no improvement in weight or reps for 3+ consecutive workouts

### 3. UserAuthentication

      concept UserAuthentication

      purpose manage user identity and session access

      principle
        after registering with credentials,
        users can authenticate and access their personal workout data

      state
        a set of Users with
          a username String
          a passwordHash String

      actions
        register (username: String, password: String): (user: User)
          requires username not already taken
          effects create new user with hashed password

        authenticate (username: String, password: String): (user: User)
          requires user exists with matching credentials
          effects return authenticated user

        deleteAccount (user: User)
          requires user exists
          effects remove user from system

### Some Synchronizations 

      sync logWorkoutSet
        when 
          UserAuthentication.authenticate (): (user)
          Request.logSet (exercise, weight, reps)
        then WorkoutLog.logSet (user, exercise, weight, reps)

      sync showSmartDefaults
        when
          UserAuthentication.authenticate (): (user)
          Request.startWorkout (exercise)
        then 
          WorkoutLog.getLastWorkout (user, exercise): (weight, reps, date)
          ProgressionGuidance.getRecommendation (user, exercise): (suggestedWeight, suggestedReps, reasoning)

      sync updateRecommendations
        when WorkoutLog.logSet (user, exercise)
        then
          WorkoutLog.getWorkoutHistory (user, exercise, startDate: 4WeeksAgo, endDate: today): (recentSets)
          ProgressionGuidance.generateRecommendation (user, exercise, recentSets)

**Note**: UserAuthentication controls access to the other concepts - all workout data and recommendations are scoped to individual users. WorkoutLog is
the primary data store that tracks all exercise performance over time for each user. ProgressionGuidance analyzes WorkoutLog data to generate 
recommendations, creating a feedback loop where each logged set triggers updated guidance. The sync showSmartDefaults demonstrates how the feauture 
*Smart Set Logging* works - when starting a workout, the system retrieves both past performance and current recommendations to pre-fill the logging 
interface.

## UI Sketches 
![Image](https://github.com/user-attachments/assets/356ecd4b-239f-463e-98ea-4436539180e3)

## User Journey 

Yannick has been stuck at 185lbs on bench press for four weeks. Every workout feels the same—he's putting in effort but not getting stronger. He opens RepRight on his phone and sees his home
screen showing recent exercises. He taps "START" to begin today's workout, which takes him to the workout logging screen.

On the logging screen, Yannick sees his bench press sets laid out with pre-filled weight and rep fields based on his last workout. He quickly logs three sets. After finishing his workout, 
he navigates to the Exercise History screen to review his progress. The screen displays his past workouts labeled by date, and at the bottom, a "View Recommendation" button catches his attention.
He taps it and sees the Plateau Intervention with a warning icon: his progress has stalled. The recommendation suggests a deload strategy. He dismisses it for now, wanting to think it over. 
But the next week, after another frustrating session at 185lbs, he returns to the recommendation and taps "Apply." Following the guided deload and progression plan over the next few weeks, 
Yannick breaks through to 195lbs. The history screen now shows clear upward progress which makes him very happy.
