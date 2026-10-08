# Windows Demos

These are demos to show Microsoft automation. Some of these are meant to be standalone and others are intended to be ran as part of a workflow. I typically use the Product Demos EE for the Job Templates.

## Setup Windows System Banner

Ansible playbook and role to install and configure [Microsoft SystemBanner](https://github.com/awawrzyniak10/SystemBanner) on Windows targets. Designed for use with Ansible Automation Platform (AAP) 2.7.

SystemBanner displays persistent security classification banners across all monitors on Windows systems, supporting both predefined classification levels (UNCLASSIFIED through TOP SECRET SCI) and fully custom text/color configurations.

### Variables

| Variable | Default | Description |
|---|---|---|
| `system_banner_state` | `present` | `present` to install/configure, `absent` to remove |
| `system_banner_version` | `v1.1.0000` | Release version tag |
| `system_banner_msi_source` | GitHub release URL | URL or local/UNC path to the MSI installer |
| `system_banner_classification` | `UNCLASSIFIED` | `UNCLASSIFIED`, `CUI`, `CONFIDENTIAL`, `SECRET`, `TOP SECRET`, `TOP SECRET SCI`, or `CUSTOM` |
| `system_banner_custom_text` | `""` | Banner text (only when classification is `CUSTOM`) |
| `system_banner_position` | `top` | `top` or `top_and_bottom` |
| `system_banner_bg_color` | `#007A33` | Background hex color (only when classification is `CUSTOM`) |
| `system_banner_fg_color` | `#FFFFFF` | Text hex color (only when classification is `CUSTOM`) |

### Manual Survey Configuration

This is configured as CaSC. For one-off, use the below information to properly configure the survey questions.

#### Question 1: Server Name or Pattern

| Field | Value |
|---|---|
| **Name** | Server Name or Pattern |
| **Description** | Specify hosts to target |
| **Type** | Text |
| **Variable** | `_hosts` |
| **Required** | Yes |


#### Question 2: Banner State

| Field | Value |
|---|---|
| **Name** | Banner State |
| **Description** | Install/configure or remove SystemBanner |
| **Type** | Multiple Choice (single select) |
| **Variable** | `system_banner_state` |
| **Required** | Yes |
| **Default** | `present` |
| **Choices** | `present`, `absent` |

#### Question 3: Classification Level

| Field | Value |
|---|---|
| **Name** | Classification Level |
| **Description** | Predefined classification marking. Choose CUSTOM to specify your own text and colors. |
| **Type** | Multiple Choice (single select) |
| **Variable** | `system_banner_classification` |
| **Required** | Yes |
| **Default** | `UNCLASSIFIED` |
| **Choices** | `UNCLASSIFIED`, `CUI`, `CONFIDENTIAL`, `SECRET`, `TOP SECRET`, `TOP SECRET SCI`, `CUSTOM` |

#### Question 4: Custom Banner Text

| Field | Value |
|---|---|
| **Name** | Custom Banner Text |
| **Description** | Text displayed on the banner (only used when Classification Level is CUSTOM) |
| **Type** | Text |
| **Variable** | `system_banner_custom_text` |
| **Required** | No |
| **Default** | *(empty)* |
| **Min Length** | 0 |
| **Max Length** | 1024 |

#### Question 5: Banner Position

| Field | Value |
|---|---|
| **Name** | Banner Position |
| **Description** | Display the banner at the top of the screen only, or at both top and bottom |
| **Type** | Multiple Choice (single select) |
| **Variable** | `system_banner_position` |
| **Required** | Yes |
| **Default** | `top` |
| **Choices** | `top`, `top_and_bottom` |

#### Question 6: Background Color

| Field | Value |
|---|---|
| **Name** | Background Color |
| **Description** | Banner background color as hex code (only used when Classification Level is CUSTOM) |
| **Type** | Text |
| **Variable** | `system_banner_bg_color` |
| **Required** | No |
| **Default** | `#007A33` |
| **Min Length** | 4 |
| **Max Length** | 7 |

#### Question 7: Text Color

| Field | Value |
|---|---|
| **Name** | Text Color |
| **Description** | Banner text/foreground color as hex code (only used when Classification Level is CUSTOM) |
| **Type** | Text |
| **Variable** | `system_banner_fg_color` |
| **Required** | No |
| **Default** | `#FFFFFF` |
| **Min Length** | 4 |
| **Max Length** | 7 |