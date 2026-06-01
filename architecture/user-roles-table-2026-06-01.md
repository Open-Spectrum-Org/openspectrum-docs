# OpenSpectrum - User Roles, Permissions, and Content Visibility Model

## Overview

OpenSpectrum is a family-centered support platform for neurodivergent children. Multiple trusted individuals may collaborate around a child, including parents, caregivers, grandparents, therapists, teachers, and clinicians.

The authorization model intentionally separates:

1. Administrative permissions
2. Content visibility permissions

This avoids creating a large number of specialized user roles.

---

# Core Concepts

## Family

A Family is the primary ownership unit within the system.

A family may contain:

* One or more children
* One or more parents
* Additional caregivers
* Therapists
* Teachers
* Medical professionals

All users belong to one or more families.

---

# Administrative Roles

Administrative roles determine what actions a user can perform within a family.

## Owner

The Owner has full control over the family account.

Permissions:

* Invite users
* Remove users
* Change user roles
* Change visibility settings
* Manage children
* Delete records
* Delete family account

Typical users:

* Parent
* Legal guardian

---

## Editor

Editors can contribute and modify content but cannot administer the family.

Permissions:

* Create records
* Edit records
* Upload attachments
* Create observations
* Create journal entries
* View permitted content

Cannot:

* Invite users
* Remove users
* Change permissions
* Delete family

Typical users:

* Parent
* Grandparent
* Therapist
* Caregiver

---

## Viewer

View-only access.

Permissions:

* View content they have permission to access

Cannot:

* Create records
* Edit records
* Delete records
* Manage users

Typical users:

* Extended family
* Temporary caregivers
* Consultants

---

# User Types

User Types are labels and do not directly grant permissions.

They are primarily used for display, filtering, and visibility rules.

Supported types:

* Parent
* Grandparent
* Caregiver
* Therapist
* Doctor
* Teacher
* Other

Future user types may be added without changing the permission system.

---

# Content Visibility Model

Every content item must contain a visibility_level field.

Visibility determines who can view the content.

Administrative role and visibility are separate concepts.

---

## Visibility: Family

Visible to all family members.

Examples:

* Activity logs
* Daily routines
* Achievement tracking
* Shared observations

---

## Visibility: Parents

Visible only to parents and legal guardians.

Examples:

* Family concerns
* Sensitive behavioral notes
* Parent discussions

---

## Visibility: Clinical

Visible to parents and clinical professionals.

Examples:

* Therapy observations
* Clinical reports
* Assessments
* Intervention notes

Users typically included:

* Parents
* Therapists
* Doctors

Users typically excluded:

* Grandparents
* Casual caregivers

---

## Visibility: Private

Visible only to the creator.

Examples:

* Draft notes
* Personal observations
* Temporary documentation

---

## Visibility: Custom

Visible only to explicitly selected users.

Used for special cases.

Examples:

* Sharing a note with a specific therapist
* Sharing a report with a specific grandparent

---

# Recommended Database Model

## users

* id
* email
* display_name
* user_type

---

## families

* id
* family_name
* created_at

---

## family_members

* id
* family_id
* user_id
* role
* joined_at

role values:

* owner
* editor
* viewer

---

## children

* id
* family_id
* name
* birth_date
* profile_data

---

## observations

* id
* child_id
* created_by
* title
* content
* visibility_level
* created_at
* updated_at

visibility_level values:

* family
* parents
* clinical
* private
* custom

---

## observation_access

Used only when visibility_level = custom

Fields:

* observation_id
* user_id

---

# Design Principles

1. Keep administrative permissions simple.
2. Keep visibility separate from roles.
3. Default new content to Family visibility.
4. Allow users to override visibility when creating content.
5. Avoid creating many specialized roles.
6. Use visibility levels to handle most privacy requirements.
7. Support future expansion without schema redesign.
