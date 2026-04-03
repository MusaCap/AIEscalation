# Data Model
## AI-Powered Call Triage & Intelligent Routing Platform

---

## 1. Overview

This data model defines the core entities, relationships, operational records, analytics structures, and governance objects required to support an AI-powered communications platform that:

- Ingests inbound voice interactions
- Transcribes and classifies requests in real time
- Routes interactions between AI agents and human analysts
- Supports enterprise and healthcare workflows
- Captures learning signals for continuous improvement
- Maintains compliance, observability, and auditability

The model is designed to support:
- Real-time operations
- Post-call reporting
- AI training and evaluation
- Human-in-the-loop decisioning
- Multi-tenant deployments
- Multi-channel expansion beyond voice

---

## 2. Design Principles

### 2.1 Core Principles
- **Channel-agnostic interaction model** so voice, chat, SMS, and future channels share a common structure
- **Strong separation of operational, analytical, and training data**
- **Event-driven history** for traceability and replay
- **Healthcare and enterprise compatibility**
- **Privacy-by-design**, with data minimization and field-level access controls
- **Human + AI collaboration as a first-class concept**

### 2.2 Architectural Data Domains
The data model is organized into these major domains:

1. Tenant & Organization
2. Identity & Users
3. End User / Caller Profiles
4. Communication Sessions
5. Media, Transcript & Conversation Understanding
6. Routing, Escalation & Assignment
7. AI Agent Execution
8. Analyst Workflow & Decision Support
9. Knowledge, Integrations & Workflows
10. Feedback, Learning & Model Governance
11. Compliance, Security & Audit
12. Analytics & Reporting

---

## 3. Entity Relationship Summary

At a high level:

- A **Tenant** has many **BusinessUnits**, **Users**, **Queues**, **Policies**, and **IntegrationConnections**
- An **EndUserProfile** can initiate many **InteractionSessions**
- An **InteractionSession** contains one or more **ConversationSegments**, **TranscriptEntries**, **IntentPredictions**, **RoutingDecisions**, and **CaseRecords**
- A session may be handled by an **AIAgent**, a **HumanAnalyst**, or both
- A **RoutingDecision** may create an **EscalationEvent**
- A **CaseRecord** tracks work performed to resolve the issue
- **FeedbackRecords** and **OutcomeRecords** feed the learning system
- **AuditEvents** record access, decisioning, workflow actions, and policy-sensitive events

---

## 4. Core Entities

---

## 4.1 Tenant

Represents a logical customer organization using the platform.

### Fields
| Field | Type | Description |
|---|---|---|
| tenant_id | UUID | Primary key |
| tenant_name | string | Organization name |
| tenant_type | enum | `enterprise`, `healthcare_provider`, `payer`, `support_center`, `hybrid` |
| status | enum | `active`, `inactive`, `suspended` |
| primary_region | string | Primary hosting/data region |
| compliance_profile_id | UUID | Associated compliance profile |
| created_at | timestamp | Creation time |
| updated_at | timestamp | Last updated |

### Relationships
- One tenant has many users
- One tenant has many business units
- One tenant has many queues
- One tenant has many integrations
- One tenant has many interaction sessions

---

## 4.2 BusinessUnit

Represents a department, service line, call center group, clinic, or customer service function.

### Fields
| Field | Type | Description |
|---|---|---|
| business_unit_id | UUID | Primary key |
| tenant_id | UUID | FK to Tenant |
| name | string | Unit name |
| unit_type | enum | `it_support`, `hr`, `billing`, `patient_access`, `nurse_triage`, `customer_support`, `operations` |
| timezone | string | Business unit timezone |
| operating_hours | JSON | Standard support windows |
| escalation_policy_id | UUID | Default escalation policy |
| created_at | timestamp | Creation time |
| updated_at | timestamp | Last updated |

---

## 4.3 ComplianceProfile

Defines regulatory constraints by tenant or business unit.

### Fields
| Field | Type | Description |
|---|---|---|
| compliance_profile_id | UUID | Primary key |
| tenant_id | UUID | FK to Tenant |
| hipaa_enabled | boolean | Whether HIPAA controls apply |
| pii_controls_enabled | boolean | Whether PII controls apply |
| phi_controls_enabled | boolean | Whether PHI controls apply |
| gdpr_enabled | boolean | Whether GDPR controls apply |
| retention_policy_days | integer | Default retention period |
| redaction_policy_id | UUID | Default redaction rules |
| consent_required_for_recording | boolean | Whether consent is required |
| created_at | timestamp | Creation time |
| updated_at | timestamp | Last updated |

---

## 5. Identity & Access Domain

---

## 5.1 User

Represents a platform operator, analyst, manager, administrator, supervisor, QA reviewer, or ML operator.

### Fields
| Field | Type | Description |
|---|---|---|
| user_id | UUID | Primary key |
| tenant_id | UUID | FK to Tenant |
| business_unit_id | UUID | FK to BusinessUnit |
| first_name | string | First name |
| last_name | string | Last name |
| email | string | Unique work email |
| user_type | enum | `analyst`, `supervisor`, `admin`, `ops_manager`, `qa_reviewer`, `ml_admin`, `integration_service` |
| status | enum | `active`, `inactive`, `locked` |
| last_login_at | timestamp | Last login |
| created_at | timestamp | Creation time |
| updated_at | timestamp | Last updated |

---

## 5.2 Role

Defines coarse-grained access roles.

### Fields
| Field | Type | Description |
|---|---|---|
| role_id | UUID | Primary key |
| tenant_id | UUID | FK to Tenant |
| role_name | string | Name of role |
| role_scope | enum | `tenant`, `business_unit`, `queue`, `system` |
| description | text | Human-readable description |

---

## 5.3 Permission

Defines specific product capabilities.

### Fields
| Field | Type | Description |
|---|---|---|
| permission_id | UUID | Primary key |
| permission_key | string | Example: `session.read`, `transcript.export`, `policy.edit` |
| description | text | Permission description |

---

## 5.4 UserRole

Join table between users and roles.

### Fields
| Field | Type |
|---|---|
| user_role_id | UUID |
| user_id | UUID |
| role_id | UUID |
| assigned_at | timestamp |
| assigned_by_user_id | UUID |

---

## 5.5 RolePermission

Maps permissions to roles.

### Fields
| Field | Type |
|---|---|
| role_permission_id | UUID |
| role_id | UUID |
| permission_id | UUID |

---

## 5.6 AccessPolicy

Supports fine-grained restrictions such as PHI access or queue-specific visibility.

### Fields
| Field | Type | Description |
|---|---|---|
| access_policy_id | UUID | Primary key |
| tenant_id | UUID | FK to Tenant |
| policy_name | string | Name |
| resource_type | enum | `transcript`, `session`, `case`, `integration_data`, `recording` |
| condition_expression | JSON | Rules logic |
| effect | enum | `allow`, `deny`, `mask` |
| created_at | timestamp | Creation time |

---

## 6. End User / Caller Domain

---

## 6.1 EndUserProfile

Represents the person contacting the system.

### Fields
| Field | Type | Description |
|---|---|---|
| end_user_id | UUID | Primary key |
| tenant_id | UUID | FK to Tenant |
| external_reference_id | string | CRM, EHR, HRIS, or customer ID |
| end_user_type | enum | `employee`, `patient`, `customer`, `vendor`, `unknown` |
| first_name | string | Optional |
| last_name | string | Optional |
| preferred_name | string | Optional |
| date_of_birth | date | Optional, highly restricted |
| phone_number | string | Primary contact |
| email | string | Optional |
| language_preference | string | Preferred language |
| accessibility_needs | JSON | Accessibility metadata |
| consent_status | enum | `unknown`, `granted`, `declined`, `revoked` |
| risk_flags | JSON | High-level risk markers |
| created_at | timestamp | Creation time |
| updated_at | timestamp | Last updated |

### Notes
In healthcare contexts, this entity may link to external patient identity systems rather than storing full medical identity locally.

---

## 6.2 EndUserIdentifier

Stores multiple identifiers for matching and deduplication.

### Fields
| Field | Type |
|---|---|
| end_user_identifier_id | UUID |
| end_user_id | UUID |
| identifier_type | enum |
| identifier_value | string |
| source_system | string |
| is_verified | boolean |
| created_at | timestamp |

### Example Identifier Types
- `mrn`
- `employee_id`
- `crm_contact_id`
- `policy_id`
- `phone_number`
- `email`

---

## 6.3 EndUserRelationship

Used when a caller is acting on behalf of another person.

### Fields
| Field | Type |
|---|---|
| relationship_id | UUID |
| primary_end_user_id | UUID |
| related_end_user_id | UUID |
| relationship_type | enum |
| authorization_status | enum |
| valid_from | timestamp |
| valid_to | timestamp |

### Example Relationship Types
- `parent_guardian`
- `caregiver`
- `spouse`
- `authorized_representative`
- `manager`
- `delegate`

---

## 7. Communication Session Domain

---

## 7.1 InteractionSession

The central record for an inbound communication event across voice or future channels.

### Fields
| Field | Type | Description |
|---|---|---|
| session_id | UUID | Primary key |
| tenant_id | UUID | FK to Tenant |
| business_unit_id | UUID | FK to BusinessUnit |
| channel_type | enum | `voice`, `chat`, `sms`, `email`, `web_message` |
| direction | enum | `inbound`, `outbound` |
| session_status | enum | `initiated`, `queued`, `active`, `transferred`, `resolved`, `abandoned`, `failed` |
| session_start_at | timestamp | Start time |
| session_end_at | timestamp | End time |
| source_system | string | Telephony or channel provider |
| source_session_id | string | Provider session ID |
| caller_end_user_id | UUID | FK to EndUserProfile |
| authenticated_end_user_id | UUID | FK to EndUserProfile |
| current_owner_type | enum | `ai_agent`, `human_analyst`, `queue`, `workflow` |
| current_owner_id | UUID | Owner reference |
| priority_level | enum | `low`, `normal`, `high`, `critical` |
| urgency_score | decimal | Computed urgency |
| sensitivity_score | decimal | Computed sensitivity |
| language_detected | string | Detected language |
| sentiment_score | decimal | Aggregate sentiment |
| risk_level | enum | `none`, `low`, `medium`, `high`, `critical` |
| resolution_status | enum | `unresolved`, `resolved_by_ai`, `resolved_by_human`, `resolved_hybrid`, `dropped` |
| resolution_code | string | Final resolution category |
| case_id | UUID | Optional FK to CaseRecord |
| created_at | timestamp | Creation time |
| updated_at | timestamp | Last updated |

### Notes
This is the top-level object used across routing, transcripts, AI actions, and analytics.

---

## 7.2 SessionParticipant

Represents any participant in the interaction.

### Fields
| Field | Type |
|---|---|
| session_participant_id | UUID |
| session_id | UUID |
| participant_type | enum |
| participant_id | UUID |
| display_name | string |
| role_in_session | enum |
| joined_at | timestamp |
| left_at | timestamp |

### Participant Types
- `end_user`
- `ai_agent`
- `human_analyst`
- `supervisor`
- `interpreter`
- `external_provider`

### Session Roles
- `caller`
- `handler`
- `observer`
- `escalation_target`

---

## 7.3 SessionChannelMetadata

Stores provider/channel-specific technical metadata.

### Fields
| Field | Type |
|---|---|
| session_channel_metadata_id | UUID |
| session_id | UUID |
| sip_call_id | string |
| telephony_provider | string |
| dialed_number | string |
| originating_number | string |
| queue_entry_at | timestamp |
| answered_at | timestamp |
| hold_duration_seconds | integer |
| transfer_count | integer |
| disconnect_reason | string |
| raw_metadata | JSON |

---

## 7.4 ConversationSegment

Represents a portion of the interaction where ownership or state changes.

### Fields
| Field | Type |
|---|---|
| segment_id | UUID |
| session_id | UUID |
| segment_type | enum |
| started_at | timestamp |
| ended_at | timestamp |
| handler_type | enum |
| handler_id | UUID |
| handoff_from_segment_id | UUID |
| summary | text |

### Segment Types
- `ivr`
- `ai_handling`
- `human_handling`
- `hold`
- `conference`
- `post_call_wrap_up`

---

## 8. Media & Transcript Domain

---

## 8.1 MediaAsset

Stores references to voice recordings or other communication media.

### Fields
| Field | Type |
|---|---|
| media_asset_id | UUID |
| session_id | UUID |
| segment_id | UUID |
| media_type | enum |
| storage_uri | string |
| storage_provider | string |
| codec | string |
| duration_seconds | integer |
| checksum | string |
| encryption_key_reference | string |
| redaction_status | enum |
| retention_expiry_at | timestamp |
| created_at | timestamp |

### Media Types
- `call_audio`
- `voicemail`
- `screen_capture`
- `attachment`

---

## 8.2 Transcript

Top-level transcript container for a session or segment.

### Fields
| Field | Type |
|---|---|
| transcript_id | UUID |
| session_id | UUID |
| segment_id | UUID |
| transcript_version | integer |
| language | string |
| transcript_status | enum |
| generated_by_model_id | UUID |
| created_at | timestamp |
| finalized_at | timestamp |

### Transcript Status
- `streaming`
- `partial`
- `final`
- `corrected`
- `redacted`

---

## 8.3 TranscriptEntry

The line-by-line or utterance-level transcript.

### Fields
| Field | Type |
|---|---|
| transcript_entry_id | UUID |
| transcript_id | UUID |
| sequence_number | integer |
| speaker_type | enum |
| speaker_id | UUID |
| start_offset_ms | integer |
| end_offset_ms | integer |
| raw_text | text |
| normalized_text | text |
| confidence_score | decimal |
| contains_pii | boolean |
| contains_phi | boolean |
| redacted_text | text |
| emotion_label | enum |
| created_at | timestamp |

### Emotion Labels
- `neutral`
- `frustrated`
- `angry`
- `confused`
- `fearful`
- `distressed`
- `positive`

---

## 8.4 EntityExtraction

Captured entities from transcript or message content.

### Fields
| Field | Type |
|---|---|
| entity_extraction_id | UUID |
| session_id | UUID |
| transcript_entry_id | UUID |
| entity_type | enum |
| entity_value | string |
| normalized_value | string |
| confidence_score | decimal |
| source_span_start | integer |
| source_span_end | integer |
| created_at | timestamp |

### Entity Types
- `appointment_date`
- `symptom`
- `location`
- `employee_id`
- `ticket_number`
- `claim_number`
- `medication`
- `department`
- `urgency_indicator`
- `contact_method`

---

## 8.5 ConversationSummary

Structured AI-generated summaries at different stages.

### Fields
| Field | Type |
|---|---|
| conversation_summary_id | UUID |
| session_id | UUID |
| segment_id | UUID |
| summary_type | enum |
| summary_text | text |
| summary_json | JSON |
| generated_by_model_id | UUID |
| confidence_score | decimal |
| visible_to_roles | JSON |
| created_at | timestamp |

### Summary Types
- `live_brief`
- `handoff_summary`
- `post_call_summary`
- `supervisor_summary`
- `compliance_summary`

---

## 9. Understanding & Classification Domain

---

## 9.1 IntentCatalog

Master list of supported intents.

### Fields
| Field | Type |
|---|---|
| intent_id | UUID |
| tenant_id | UUID |
| intent_key | string |
| display_name | string |
| description | text |
| intent_domain | enum |
| resolution_type | enum |
| risk_category | enum |
| active_flag | boolean |
| created_at | timestamp |
| updated_at | timestamp |

### Intent Domains
- `it_support`
- `hr`
- `benefits`
- `billing`
- `appointment`
- `clinical_triage`
- `customer_support`
- `general_inquiry`

### Resolution Types
- `ai_resolvable`
- `human_required`
- `conditional`
- `emergency_escalation`

---

## 9.2 IntentPrediction

Stores model predictions over the course of the session.

### Fields
| Field | Type |
|---|---|
| intent_prediction_id | UUID |
| session_id | UUID |
| segment_id | UUID |
| transcript_entry_id | UUID |
| predicted_intent_id | UUID |
| model_id | UUID |
| confidence_score | decimal |
| rank_order | integer |
| prediction_stage | enum |
| explanation_json | JSON |
| created_at | timestamp |

### Prediction Stages
- `early_streaming`
- `mid_session`
- `pre_handoff`
- `post_session_final`

---

## 9.3 TopicClassification

Tracks broader conversation topics beyond discrete intent.

### Fields
| Field | Type |
|---|---|
| topic_classification_id | UUID |
| session_id | UUID |
| topic_name | string |
| confidence_score | decimal |
| created_at | timestamp |

---

## 9.4 RiskAssessment

Tracks operational, clinical, legal, or emotional risk signals.

### Fields
| Field | Type |
|---|---|
| risk_assessment_id | UUID |
| session_id | UUID |
| assessment_type | enum |
| risk_level | enum |
| risk_score | decimal |
| signal_source | enum |
| detected_signal | string |
| policy_triggered | boolean |
| recommended_action | enum |
| created_at | timestamp |

### Assessment Types
- `clinical_urgency`
- `self_harm_risk`
- `abuse_indicator`
- `legal_sensitive`
- `privacy_sensitive`
- `escalation_complexity`
- `fraud_risk`

### Recommended Actions
- `continue_ai`
- `ask_clarifying_question`
- `route_to_specialist`
- `immediate_human_takeover`
- `emergency_instruction`

---

## 9.5 SentimentAssessment

Tracks sentiment changes over time.

### Fields
| Field | Type |
|---|---|
| sentiment_assessment_id | UUID |
| session_id | UUID |
| segment_id | UUID |
| sentiment_label | enum |
| sentiment_score | decimal |
| trajectory | enum |
| created_at | timestamp |

### Trajectory
- `improving`
- `stable`
- `worsening`

---

## 10. Routing & Escalation Domain

---

## 10.1 RoutingPolicy

Configurable policy that governs AI-vs-human decisions.

### Fields
| Field | Type |
|---|---|
| routing_policy_id | UUID |
| tenant_id | UUID |
| business_unit_id | UUID |
| policy_name | string |
| channel_type | enum |
| intent_id | UUID |
| minimum_ai_confidence | decimal |
| maximum_allowed_risk | enum |
| requires_human_if_sensitive | boolean |
| requires_human_if_new_intent | boolean |
| requires_human_for_first_contact | boolean |
| after_hours_behavior | enum |
| active_flag | boolean |
| created_at | timestamp |
| updated_at | timestamp |

---

## 10.2 RoutingDecision

Stores each routing decision made during a session.

### Fields
| Field | Type |
|---|---|
| routing_decision_id | UUID |
| session_id | UUID |
| segment_id | UUID |
| decision_timestamp | timestamp |
| decision_type | enum |
| decided_by | enum |
| policy_id | UUID |
| primary_intent_id | UUID |
| confidence_score | decimal |
| risk_level | enum |
| recommended_target_type | enum |
| recommended_target_id | UUID |
| final_target_type | enum |
| final_target_id | UUID |
| decision_reason | text |
| decision_reason_json | JSON |
| overridden_by_user_id | UUID |
| override_reason | text |

### Decision Types
- `initial_route`
- `reroute`
- `escalation`
- `deescalation`
- `specialist_transfer`

### Decided By
- `rule_engine`
- `classification_model`
- `supervisor_override`
- `analyst_override`
- `hybrid_decision_service`

---

## 10.3 Queue

Represents an assignment queue for analysts or specialists.

### Fields
| Field | Type |
|---|---|
| queue_id | UUID |
| tenant_id | UUID |
| business_unit_id | UUID |
| queue_name | string |
| queue_type | enum |
| skill_tags | JSON |
| priority_rules | JSON |
| max_wait_seconds | integer |
| active_flag | boolean |
| created_at | timestamp |

### Queue Types
- `general_support`
- `clinical_triage`
- `billing`
- `vip_support`
- `after_hours`
- `escalations`

---

## 10.4 QueueAssignment

Links a session to a queue over time.

### Fields
| Field | Type |
|---|---|
| queue_assignment_id | UUID |
| session_id | UUID |
| queue_id | UUID |
| assigned_at | timestamp |
| accepted_at | timestamp |
| released_at | timestamp |
| assignment_status | enum |
| priority_score | decimal |

### Assignment Status
- `pending`
- `accepted`
- `expired`
- `reassigned`
- `completed`

---

## 10.5 EscalationEvent

Represents an AI-to-human or human-to-specialist escalation.

### Fields
| Field | Type |
|---|---|
| escalation_event_id | UUID |
| session_id | UUID |
| from_handler_type | enum |
| from_handler_id | UUID |
| to_handler_type | enum |
| to_handler_id | UUID |
| escalation_reason_code | string |
| escalation_reason_text | text |
| escalation_priority | enum |
| escalation_trigger_source | enum |
| handoff_summary_id | UUID |
| created_at | timestamp |
| acknowledged_at | timestamp |
| completed_at | timestamp |

### Trigger Source
- `confidence_threshold`
- `risk_policy`
- `caller_request`
- `analyst_request`
- `workflow_failure`
- `integration_failure`

---

## 11. AI Agent Domain

---

## 11.1 AIAgent

Defines a deployable AI agent persona and capability set.

### Fields
| Field | Type |
|---|---|
| ai_agent_id | UUID |
| tenant_id | UUID |
| agent_name | string |
| agent_type | enum |
| description | text |
| supported_channels | JSON |
| supported_intents | JSON |
| policy_bundle_id | UUID |
| active_model_id | UUID |
| active_flag | boolean |
| created_at | timestamp |
| updated_at | timestamp |

### Agent Types
- `general_triage`
- `it_support`
- `billing_assistant`
- `appointment_assistant`
- `patient_access_assistant`
- `faq_agent`

---

## 11.2 AIAgentSession

Tracks the AI agent's participation in a session.

### Fields
| Field | Type |
|---|---|
| ai_agent_session_id | UUID |
| session_id | UUID |
| ai_agent_id | UUID |
| started_at | timestamp |
| ended_at | timestamp |
| termination_reason | enum |
| resolution_outcome | enum |
| fallback_triggered | boolean |

### Termination Reasons
- `resolved`
- `escalated`
- `timeout`
- `handoff_requested`
- `policy_stop`
- `system_error`

---

## 11.3 AIResponse

Stores generated or selected AI responses.

### Fields
| Field | Type |
|---|---|
| ai_response_id | UUID |
| session_id | UUID |
| segment_id | UUID |
| prompt_context_id | UUID |
| response_type | enum |
| response_text | text |
| response_json | JSON |
| response_confidence | decimal |
| grounded_source_refs | JSON |
| safety_review_status | enum |
| sent_to_user | boolean |
| created_at | timestamp |

### Response Types
- `spoken_response`
- `clarifying_question`
- `instruction`
- `workflow_confirmation`
- `handoff_message`
- `silent_suggestion`

---

## 11.4 PromptContext

Captures the context used for AI generation.

### Fields
| Field | Type |
|---|---|
| prompt_context_id | UUID |
| session_id | UUID |
| system_prompt_version | string |
| retrieved_knowledge_refs | JSON |
| user_context_snapshot | JSON |
| conversation_window | JSON |
| policy_constraints | JSON |
| created_at | timestamp |

---

## 11.5 ToolInvocation

Tracks external tools/workflows called by the AI.

### Fields
| Field | Type |
|---|---|
| tool_invocation_id | UUID |
| session_id | UUID |
| ai_response_id | UUID |
| tool_name | string |
| invocation_type | enum |
| request_payload | JSON |
| response_payload | JSON |
| invocation_status | enum |
| latency_ms | integer |
| error_code | string |
| error_message | text |
| created_at | timestamp |

### Invocation Types
- `api_call`
- `workflow_trigger`
- `knowledge_lookup`
- `identity_verification`
- `record_update`

---

## 12. Human Analyst Workflow Domain

---

## 12.1 AnalystProfile

Operational profile for an analyst.

### Fields
| Field | Type |
|---|---|
| analyst_profile_id | UUID |
| user_id | UUID |
| analyst_level | enum |
| skill_tags | JSON |
| certification_tags | JSON |
| languages_supported | JSON |
| max_concurrent_sessions | integer |
| active_shift_status | enum |
| created_at | timestamp |
| updated_at | timestamp |

### Analyst Levels
- `tier_1`
- `tier_2`
- `specialist`
- `clinical_reviewer`
- `supervisor`

---

## 12.2 AnalystAssignment

Tracks a human taking ownership of a session or case.

### Fields
| Field | Type |
|---|---|
| analyst_assignment_id | UUID |
| session_id | UUID |
| analyst_user_id | UUID |
| assigned_at | timestamp |
| accepted_at | timestamp |
| released_at | timestamp |
| assignment_reason | enum |
| workload_snapshot | JSON |

### Assignment Reasons
- `escalation`
- `manual_pickup`
- `supervisor_assignment`
- `overflow`
- `follow_up`

---

## 12.3 AnalystAction

Logs actions performed in the dashboard.

### Fields
| Field | Type |
|---|---|
| analyst_action_id | UUID |
| session_id | UUID |
| analyst_user_id | UUID |
| action_type | enum |
| action_payload | JSON |
| source_suggestion_id | UUID |
| created_at | timestamp |

### Action Types
- `send_response`
- `select_suggested_reply`
- `update_case`
- `trigger_workflow`
- `override_intent`
- `override_routing`
- `mark_priority`
- `request_supervisor`
- `resolve_session`

---

## 12.4 DecisionSupportSuggestion

AI-generated suggestions shown to analysts.

### Fields
| Field | Type |
|---|---|
| suggestion_id | UUID |
| session_id | UUID |
| suggestion_type | enum |
| suggestion_text | text |
| suggestion_payload | JSON |
| source_basis | JSON |
| confidence_score | decimal |
| acknowledged_by_user_id | UUID |
| accepted_flag | boolean |
| rejected_reason | string |
| created_at | timestamp |

### Suggestion Types
- `recommended_reply`
- `recommended_workflow`
- `recommended_transfer`
- `recommended_question`
- `knowledge_article`
- `compliance_warning`

---

## 12.5 WrapUpRecord

Post-call structured completion record.

### Fields
| Field | Type |
|---|---|
| wrap_up_record_id | UUID |
| session_id | UUID |
| analyst_user_id | UUID |
| wrap_up_code | string |
| disposition | enum |
| follow_up_required | boolean |
| follow_up_due_at | timestamp |
| notes | text |
| created_at | timestamp |

### Dispositions
- `resolved`
- `pending_follow_up`
- `referred`
- `unable_to_reach`
- `duplicate`
- `escalated_out`

---

## 13. Case Management Domain

---

## 13.1 CaseRecord

Represents the problem instance being resolved. A session may create or attach to a case.

### Fields
| Field | Type |
|---|---|
| case_id | UUID |
| tenant_id | UUID |
| business_unit_id | UUID |
| end_user_id | UUID |
| originating_session_id | UUID |
| case_type | enum |
| status | enum |
| severity | enum |
| owner_type | enum |
| owner_id | UUID |
| opened_at | timestamp |
| resolved_at | timestamp |
| closed_at | timestamp |
| final_resolution_code | string |
| created_at | timestamp |
| updated_at | timestamp |

### Case Types
- `support_request`
- `billing_issue`
- `appointment_request`
- `clinical_review`
- `complaint`
- `incident`
- `benefits_question`

---

## 13.2 CaseEvent

Timeline event within the case lifecycle.

### Fields
| Field | Type |
|---|---|
| case_event_id | UUID |
| case_id | UUID |
| event_type | enum |
| event_payload | JSON |
| performed_by_type | enum |
| performed_by_id | UUID |
| created_at | timestamp |

### Event Types
- `case_opened`
- `session_linked`
- `status_changed`
- `note_added`
- `workflow_triggered`
- `owner_changed`
- `resolved`
- `closed`

---

## 13.3 CaseNote

Human- or AI-authored notes related to a case.

### Fields
| Field | Type |
|---|---|
| case_note_id | UUID |
| case_id | UUID |
| author_type | enum |
| author_id | UUID |
| note_type | enum |
| note_text | text |
| visibility_scope | enum |
| created_at | timestamp |

### Visibility Scope
- `internal`
- `supervisors_only`
- `external_shareable`

---

## 14. Knowledge & Workflow Domain

---

## 14.1 KnowledgeAsset

Represents an article, policy, FAQ, playbook, or protocol.

### Fields
| Field | Type |
|---|---|
| knowledge_asset_id | UUID |
| tenant_id | UUID |
| asset_type | enum |
| title | string |
| summary | text |
| body_uri | string |
| source_system | string |
| version_label | string |
| tags | JSON |
| effective_from | timestamp |
| effective_to | timestamp |
| approval_status | enum |
| created_at | timestamp |
| updated_at | timestamp |

### Asset Types
- `faq`
- `policy`
- `clinical_protocol`
- `script`
- `playbook`
- `troubleshooting_guide`

---

## 14.2 KnowledgeRetrievalEvent

Tracks what knowledge was retrieved for AI or analysts.

### Fields
| Field | Type |
|---|---|
| knowledge_retrieval_event_id | UUID |
| session_id | UUID |
| knowledge_asset_id | UUID |
| retrieved_for | enum |
| retrieval_score | decimal |
| retrieval_rank | integer |
| used_in_response_flag | boolean |
| created_at | timestamp |

### Retrieved For
- `ai_response`
- `analyst_suggestion`
- `supervisor_review`

---

## 14.3 WorkflowDefinition

Represents a structured automation or workflow.

### Fields
| Field | Type |
|---|---|
| workflow_definition_id | UUID |
| tenant_id | UUID |
| workflow_name | string |
| workflow_type | enum |
| trigger_conditions | JSON |
| steps_definition | JSON |
| required_permissions | JSON |
| active_flag | boolean |
| created_at | timestamp |
| updated_at | timestamp |

### Workflow Types
- `identity_verification`
- `password_reset`
- `appointment_reschedule`
- `benefits_lookup`
- `ticket_creation`
- `billing_follow_up`

---

## 14.4 WorkflowExecution

Tracks runtime workflow execution.

### Fields
| Field | Type |
|---|---|
| workflow_execution_id | UUID |
| session_id | UUID |
| case_id | UUID |
| workflow_definition_id | UUID |
| initiated_by_type | enum |
| initiated_by_id | UUID |
| execution_status | enum |
| started_at | timestamp |
| completed_at | timestamp |
| output_payload | JSON |
| error_payload | JSON |

### Execution Status
- `started`
- `in_progress`
- `completed`
- `failed`
- `rolled_back`
- `awaiting_input`

---

## 15. Integration Domain

---

## 15.1 IntegrationConnection

Configured connection to an external platform.

### Fields
| Field | Type |
|---|---|
| integration_connection_id | UUID |
| tenant_id | UUID |
| integration_type | enum |
| provider_name | string |
| connection_name | string |
| auth_method | enum |
| status | enum |
| base_uri | string |
| metadata | JSON |
| created_at | timestamp |
| updated_at | timestamp |

### Integration Types
- `crm`
- `ehr`
- `hris`
- `ticketing`
- `telephony`
- `identity`
- `knowledge_base`
- `analytics_sink`

---

## 15.2 ExternalRecordLink

Maps internal entities to external records.

### Fields
| Field | Type |
|---|---|
| external_record_link_id | UUID |
| integration_connection_id | UUID |
| entity_type | enum |
| entity_id | UUID |
| external_object_type | string |
| external_object_id | string |
| sync_status | enum |
| last_synced_at | timestamp |

---

## 15.3 IntegrationEvent

Tracks inbound or outbound data exchange.

### Fields
| Field | Type |
|---|---|
| integration_event_id | UUID |
| integration_connection_id | UUID |
| session_id | UUID |
| case_id | UUID |
| event_direction | enum |
| event_type | string |
| request_payload | JSON |
| response_payload | JSON |
| status | enum |
| latency_ms | integer |
| created_at | timestamp |

### Event Direction
- `inbound`
- `outbound`

---

## 16. Feedback, Outcomes & Learning Domain

---

## 16.1 OutcomeRecord

Represents the final measurable result of the interaction.

### Fields
| Field | Type |
|---|---|
| outcome_record_id | UUID |
| session_id | UUID |
| case_id | UUID |
| outcome_type | enum |
| successful_resolution | boolean |
| first_contact_resolved | boolean |
| escalated_flag | boolean |
| reopened_within_days | integer |
| user_satisfaction_score | decimal |
| analyst_effort_score | decimal |
| created_at | timestamp |

### Outcome Types
- `resolved_ai_only`
- `resolved_human_only`
- `resolved_hybrid`
- `unresolved`
- `abandoned`

---

## 16.2 FeedbackRecord

Captures explicit human feedback.

### Fields
| Field | Type |
|---|---|
| feedback_record_id | UUID |
| session_id | UUID |
| case_id | UUID |
| feedback_source | enum |
| source_user_id | UUID |
| feedback_type | enum |
| feedback_value | string |
| comments | text |
| created_at | timestamp |

### Feedback Source
- `analyst`
- `supervisor`
- `end_user`
- `qa_team`

### Feedback Type
- `intent_correctness`
- `routing_correctness`
- `response_quality`
- `handoff_quality`
- `knowledge_relevance`
- `overall_experience`

---

## 16.3 LabelCorrection

Stores explicit corrections used for training.

### Fields
| Field | Type |
|---|---|
| label_correction_id | UUID |
| session_id | UUID |
| original_label_type | enum |
| original_label_value | string |
| corrected_label_value | string |
| corrected_by_user_id | UUID |
| correction_reason | text |
| created_at | timestamp |

### Label Types
- `intent`
- `risk`
- `priority`
- `resolution_code`
- `sentiment`

---

## 16.4 TrainingExample

Normalized examples for ML retraining.

### Fields
| Field | Type |
|---|---|
| training_example_id | UUID |
| tenant_id | UUID |
| source_session_id | UUID |
| source_transcript_id | UUID |
| example_type | enum |
| input_text | text |
| label_json | JSON |
| quality_score | decimal |
| eligibility_status | enum |
| redaction_status | enum |
| created_at | timestamp |

### Eligibility Status
- `pending_review`
- `approved`
- `rejected`
- `held_for_compliance`

---

## 16.5 ModelRegistry

Catalog of deployed and historical models.

### Fields
| Field | Type |
|---|---|
| model_id | UUID |
| model_name | string |
| model_family | enum |
| model_version | string |
| model_purpose | enum |
| training_dataset_version | string |
| deployment_status | enum |
| created_at | timestamp |
| deployed_at | timestamp |
| retired_at | timestamp |

### Model Purpose
- `stt`
- `intent_classification`
- `risk_detection`
- `response_generation`
- `summarization`
- `retrieval_ranking`

---

## 16.6 ModelEvaluationRun

Stores offline or online evaluation results.

### Fields
| Field | Type |
|---|---|
| model_evaluation_run_id | UUID |
| model_id | UUID |
| evaluation_type | enum |
| dataset_reference | string |
| metrics_json | JSON |
| evaluator_user_id | UUID |
| started_at | timestamp |
| completed_at | timestamp |

### Evaluation Type
- `offline_validation`
- `shadow_test`
- `a_b_test`
- `production_monitoring`

---

## 17. Compliance, Audit & Security Domain

---

## 17.1 ConsentRecord

Tracks recording, processing, and disclosure consent.

### Fields
| Field | Type |
|---|---|
| consent_record_id | UUID |
| end_user_id | UUID |
| session_id | UUID |
| consent_type | enum |
| consent_status | enum |
| captured_via | enum |
| evidence_uri | string |
| captured_at | timestamp |
| expires_at | timestamp |

### Consent Types
- `call_recording`
- `ai_processing`
- `data_sharing`
- `sms_follow_up`

---

## 17.2 RedactionRule

Defines redaction patterns and policies.

### Fields
| Field | Type |
|---|---|
| redaction_rule_id | UUID |
| tenant_id | UUID |
| rule_name | string |
| target_data_type | enum |
| pattern_definition | JSON |
| replacement_strategy | enum |
| active_flag | boolean |
| created_at | timestamp |

### Target Data Types
- `pii`
- `phi`
- `payment_card`
- `credentials`
- `custom_sensitive`

---

## 17.3 AuditEvent

Immutable audit trail of sensitive operations.

### Fields
| Field | Type |
|---|---|
| audit_event_id | UUID |
| tenant_id | UUID |
| actor_type | enum |
| actor_id | UUID |
| event_category | enum |
| event_action | string |
| resource_type | string |
| resource_id | UUID |
| before_state_hash | string |
| after_state_hash | string |
| ip_address | string |
| user_agent | string |
| created_at | timestamp |

### Event Categories
- `authentication`
- `authorization`
- `data_access`
- `routing_decision`
- `configuration_change`
- `export`
- `model_override`
- `compliance_event`

---

## 17.4 DataRetentionPolicy

Controls lifecycle and deletion windows.

### Fields
| Field | Type |
|---|---|
| data_retention_policy_id | UUID |
| tenant_id | UUID |
| data_domain | enum |
| retention_days | integer |
| deletion_mode | enum |
| legal_hold_supported | boolean |
| created_at | timestamp |

### Data Domains
- `audio`
- `transcript`
- `case`
- `feedback`
- `training_data`
- `audit`

---

## 18. Analytics & Reporting Domain

---

## 18.1 SessionMetricsSnapshot

Stores aggregated operational metrics at the session level.

### Fields
| Field | Type |
|---|---|
| session_metrics_snapshot_id | UUID |
| session_id | UUID |
| total_duration_seconds | integer |
| queue_wait_seconds | integer |
| ai_handling_seconds | integer |
| human_handling_seconds | integer |
| hold_seconds | integer |
| total_transfers | integer |
| total_escalations | integer |
| total_tool_invocations | integer |
| generated_at | timestamp |

---

## 18.2 QueueMetricsDaily

Daily queue rollups.

### Fields
| Field | Type |
|---|---|
| queue_metrics_daily_id | UUID |
| queue_id | UUID |
| metric_date | date |
| sessions_received | integer |
| sessions_answered | integer |
| sessions_abandoned | integer |
| avg_wait_seconds | decimal |
| avg_handle_seconds | decimal |
| ai_resolution_rate | decimal |
| sla_attainment_rate | decimal |
| generated_at | timestamp |

---

## 18.3 AnalystMetricsDaily

Daily productivity and quality rollups for analysts.

### Fields
| Field | Type |
|---|---|
| analyst_metrics_daily_id | UUID |
| analyst_user_id | UUID |
| metric_date | date |
| sessions_handled | integer |
| avg_handle_seconds | decimal |
| avg_wrap_up_seconds | decimal |
| csat_average | decimal |
| override_rate | decimal |
| first_contact_resolution_rate | decimal |
| generated_at | timestamp |

---

## 18.4 ModelPerformanceMetric

Production monitoring metrics for AI models.

### Fields
| Field | Type |
|---|---|
| model_performance_metric_id | UUID |
| model_id | UUID |
| metric_window_start | timestamp |
| metric_window_end | timestamp |
| metric_name | string |
| metric_value | decimal |
| segment_filter | JSON |
| created_at | timestamp |

### Example Metrics
- `intent_accuracy`
- `escalation_precision`
- `escalation_recall`
- `hallucination_rate`
- `handoff_acceptance_rate`
- `response_latency_p95`

---

## 19. Configuration & Policy Domain

---

## 19.1 PolicyBundle

Groups policies assigned to AI agents, business units, or tenants.

### Fields
| Field | Type |
|---|---|
| policy_bundle_id | UUID |
| tenant_id | UUID |
| bundle_name | string |
| description | text |
| active_flag | boolean |
| created_at | timestamp |

---

## 19.2 SafetyPolicy

Defines allowed/disallowed behavior for AI.

### Fields
| Field | Type |
|---|---|
| safety_policy_id | UUID |
| policy_bundle_id | UUID |
| policy_name | string |
| restricted_topic | string |
| action_on_violation | enum |
| escalation_target_queue_id | UUID |
| created_at | timestamp |

### Action On Violation
- `block_response`
- `handoff_to_human`
- `replace_with_safe_response`
- `alert_supervisor`

---

## 19.3 SLAConfiguration

Service-level expectations by queue or intent.

### Fields
| Field | Type |
|---|---|
| sla_configuration_id | UUID |
| tenant_id | UUID |
| queue_id | UUID |
| intent_id | UUID |
| target_response_seconds | integer |
| target_resolution_seconds | integer |
| breach_action | enum |
| active_flag | boolean |

---

## 20. Recommended Relational Schema Boundaries

For implementation, split storage by workload:

### 20.1 Operational OLTP Database
Best for:
- sessions
- routing decisions
- users
- queues
- cases
- workflow executions
- permissions

Likely tables:
- `tenants`
- `business_units`
- `users`
- `roles`
- `permissions`
- `end_users`
- `interaction_sessions`
- `routing_decisions`
- `queues`
- `queue_assignments`
- `escalation_events`
- `case_records`
- `workflow_executions`

### 20.2 Search / Retrieval Store
Best for:
- transcripts
- summaries
- knowledge assets
- embedding-backed semantic retrieval

Likely indexes:
- `transcripts_index`
- `knowledge_index`
- `conversation_summary_index`

### 20.3 Event / Streaming Store
Best for:
- live transcript events
- tool invocation events
- routing events
- monitoring data

Likely topics/streams:
- `session.created`
- `audio.chunk.received`
- `transcript.updated`
- `intent.predicted`
- `routing.decision.made`
- `escalation.triggered`
- `workflow.executed`
- `feedback.recorded`

### 20.4 Analytical Warehouse
Best for:
- daily metrics
- performance dashboards
- AI evaluation
- historical SLA analysis

Fact tables:
- `fact_sessions`
- `fact_transcript_events`
- `fact_routing_decisions`
- `fact_escalations`
- `fact_workflow_executions`
- `fact_feedback`
- `fact_model_inference`

Dimensions:
- `dim_tenant`
- `dim_business_unit`
- `dim_queue`
- `dim_user`
- `dim_end_user_type`
- `dim_intent`
- `dim_channel`
- `dim_model`
- `dim_time`

---

## 21. Canonical Relationships

Below are the most important one-to-many and many-to-many relationships.

### One-to-Many
- Tenant → BusinessUnits
- Tenant → Users
- Tenant → EndUserProfiles
- Tenant → InteractionSessions
- InteractionSession → ConversationSegments
- InteractionSession → TranscriptEntries
- InteractionSession → IntentPredictions
- InteractionSession → RoutingDecisions
- InteractionSession → EscalationEvents
- InteractionSession → AIResponses
- InteractionSession → AnalystActions
- InteractionSession → WorkflowExecutions
- CaseRecord → CaseEvents
- CaseRecord → CaseNotes

### Many-to-Many
- Users ↔ Roles
- Roles ↔ Permissions
- Sessions ↔ Queues through QueueAssignment
- Sessions ↔ KnowledgeAssets through KnowledgeRetrievalEvent
- Sessions ↔ Models through inference/evaluation records

---

## 22. Example End-to-End Data Flow Using the Model

### Scenario: Employee calls for password reset
1. Create `InteractionSession`
2. Match or create `EndUserProfile`
3. Stream audio into `MediaAsset`
4. Generate `Transcript` and `TranscriptEntry` records
5. Create `IntentPrediction` with `password_reset`
6. Apply `RoutingPolicy`
7. Create `RoutingDecision` = AI-resolvable
8. Instantiate `AIAgentSession`
9. Trigger `WorkflowExecution` for password reset
10. Store `AIResponse`
11. Create `OutcomeRecord` = resolved_ai_only
12. Log `SessionMetricsSnapshot`
13. Capture optional `FeedbackRecord`

### Scenario: Patient reports chest pain
1. Create `InteractionSession`
2. Capture transcript in real time
3. `IntentPrediction` = possible clinical concern
4. `RiskAssessment` = critical
5. `RoutingDecision` = escalate immediately
6. `EscalationEvent` created
7. `QueueAssignment` to clinical queue
8. `ConversationSummary` = handoff summary
9. Analyst opens case via `CaseRecord`
10. All actions logged with `AuditEvent`

---

## 23. Minimum MVP Data Model

For an MVP, the platform can launch with the following minimum entity set:

### Must-Have MVP Tables
- Tenant
- User
- AnalystProfile
- EndUserProfile
- InteractionSession
- SessionParticipant
- MediaAsset
- Transcript
- TranscriptEntry
- IntentCatalog
- IntentPrediction
- RoutingPolicy
- RoutingDecision
- Queue
- QueueAssignment
- EscalationEvent
- AIAgent
- AIAgentSession
- AIResponse
- AnalystAssignment
- AnalystAction
- ConversationSummary
- FeedbackRecord
- OutcomeRecord
- AuditEvent

### MVP Deferred / Phase 2+
- LabelCorrection
- TrainingExample
- ModelEvaluationRun
- DataRetentionPolicy
- AccessPolicy
- WorkflowDefinition / advanced automation
- Deep case management
- Advanced external record linking
- Multi-party relationship handling

---

## 24. Recommended Enumerations

To reduce inconsistency, centralize enums in a controlled configuration layer.

### Important Shared Enums
- channel_type
- session_status
- participant_type
- risk_level
- resolution_status
- intent_domain
- resolution_type
- queue_type
- feedback_type
- workflow_type
- integration_type
- model_purpose

---

## 25. Data Governance Recommendations

### 25.1 Sensitive Data Partitioning
Separate:
- identity data
- transcript content
- medical/clinical fields
- authentication evidence
- analytics rollups

### 25.2 Field-Level Protection
Apply masking or encryption to:
- phone numbers
- date of birth
- MRN / employee IDs
- transcript spans containing PHI/PII
- consent evidence
- external system identifiers

### 25.3 Audit Requirements
All of the following should generate `AuditEvent` records:
- transcript access
- recording playback
- routing override
- policy changes
- export actions
- model threshold changes
- knowledge article changes in regulated domains

---

## 26. Suggested Physical Keys and Indexes

### High-Priority Indexes
- `interaction_sessions(tenant_id, session_start_at)`
- `interaction_sessions(caller_end_user_id, session_start_at desc)`
- `transcript_entries(transcript_id, sequence_number)`
- `intent_predictions(session_id, created_at)`
- `routing_decisions(session_id, decision_timestamp)`
- `queue_assignments(queue_id, assignment_status, assigned_at)`
- `escalation_events(session_id, created_at)`
- `audit_events(resource_type, resource_id, created_at)`
- `feedback_records(session_id, created_at)`
- `workflow_executions(session_id, execution_status)`

### Search Index Candidates
- transcript normalized text
- conversation summary text
- knowledge asset title/body
- case notes text

---

## 27. Example Warehouse Star Schema

### Fact Tables
- `fact_session`
- `fact_escalation`
- `fact_ai_response`
- `fact_workflow_execution`
- `fact_feedback`
- `fact_model_prediction`

### Dimension Tables
- `dim_date`
- `dim_time`
- `dim_tenant`
- `dim_business_unit`
- `dim_queue`
- `dim_intent`
- `dim_channel`
- `dim_user`
- `dim_end_user_type`
- `dim_model`
- `dim_risk_level`

### Example Measures
- count of sessions
- count of escalations
- average AI handling time
- average human handling time
- AI resolution rate
- misroute rate
- analyst override rate
- CSAT average
- SLA breach count

---

## 28. Open Data Model Decisions

These are the main schema decisions to resolve during implementation:

1. Should `CaseRecord` be mandatory for every session, or only for escalated/high-value interactions?
2. Should transcript corrections overwrite `TranscriptEntry` or create immutable new versions?
3. How much user identity should be stored natively vs referenced from external systems?
4. Should healthcare and enterprise intents share one global taxonomy or separate taxonomies?
5. Should routing policy be rules-first, model-first, or hybrid with explicit precedence?
6. How will data residency be enforced for multi-region tenants?
7. What data is permitted to enter training sets in HIPAA-regulated environments?

---

## 29. Final Recommendation

The best implementation pattern is a **hybrid operational + event-driven model**:

- Use a relational OLTP core for sessions, routing, cases, queues, and users
- Use an event stream for real-time transcripts and decision events
- Use a search layer for transcript and knowledge retrieval
- Use a warehouse for reporting, model monitoring, and optimization
- Treat compliance, redaction, and auditability as first-class schema concerns, not add-ons

This model gives the platform the flexibility to:
- launch quickly with core voice triage
- support enterprise and healthcare needs with the same platform backbone
- expand into chat/SMS and predictive support later
- safely improve AI performance over time through structured human feedback
