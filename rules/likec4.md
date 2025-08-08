# LikeC4 Project Rules and Best Practices

This document defines some guidelines for [LikeC4](https://likec4.dev).

## 1. Overview

The content of a LikeC4 project should be as self-explanatory as possible. These projects are often the result of painstaking research, reading documentation, and interviews. Projects should reference objective sources wherever they exist, including Slack channels, wikis, SharePoint, etc.

The metadata within these projects is as useful as the models and relationships.

### Key Outputs
- **Metadata-driven**: Extensive use of metadata for traceability to source documents [models](https://likec4.dev/dsl/model/#element-metadata), [relationships](https://likec4.dev/dsl/relationships/#relationships-metadata)
- **Tag-based filtering**: Heavy use of tags for filtering views by business domain [tags](https://likec4.dev/dsl/model/#element-tags)
- **Hierarchical organization**: Model hierarchy is optional, use it if it helps your views
- **Relationship labeling**: Add data to relationships, have descriptive labels explaining the interaction
- **Source documentation**: Links to external documentation and architecture sources
- **Single/Multi-Project**: A C4 project can contain a single "project" or multiple related projects, use a common spec (with sym-links) in this case [Multi-projects](https://likec4.dev/dsl/projects/)

## 2. Project Layout

### Directory Structure

#### Single-Project
Do the simplest thing that works for you.
```
project-root/
├── _spec.c4               # Shared specifications
├── model.c4               # Main model definitions
├── views/
│   ├── view.c4            # Static view definitions
│   └── use-cases/
│       └── flow-name.c4   # Dynamic views for specific flows
└── deployment/
    ├── deployment.c4      # Main deployment diagram
    └── domains.c4         # Domain-specific deployment views
```

#### Multi-Project

Multiple projects can be areas of work or now and next versions, whatever helps explain your world.

- Each project has a `likec4.config.json` with a `name` property
    - `_linked-spec.c4` are sym-links to "share" the common specification
- Common configuration is minimal and focused on project identification

```
project-root/
├── common/
│   ├── _spec.c4               # Shared specifications
│   └── likec4.config.json     # Common project config
├── domain-project-1/
│   ├── likec4.config.json     # Project-specific config
│   ├── _linked-spec.c4        # Linked to common spec
│   └── [same as for a single project without the specification]
└── domain-project-2/
    └── [same structure as above]
```

### File Naming Conventions

- `model.[name.]c4` - Main model definitions, (name is optional if this becomes too large)
- `_spec.c4` - Specification file
- `view.[name.]c4` - View definitions, (name is optional if this becomes too large)
- `deployment.c4` - Deployment diagrams
- `domains.c4` - Domain-specific deployment views
- `use-cases/*.c4` - Dynamic views for specific flows
- Multi-projects require a `likec4.config.json` with a `name` property see [Multi-projects](https://likec4.dev/dsl/projects/)

It is perfectly valid to split out the `_spec.c4` file for tags, relationships, elements, etc. if the size warrants it.

## 3. Specification Files

### Specification Structure
The specification defines the vocabulary and visual styling for the C4 model. Where quotes are required, be consistent and use single quotes.

### Basic Naming
- Use lowercase LikeC4 DSL entities
- Use meaningful names for elements, relationships, and entities
- Use a consistent convention
- Use consistent word separators, pick one (`order-manager`, `order_manager`, `orderManager`), you may find camelCase works better with your IDE
- Keep names brief but meaningful, other attributes can provide a fuller description
- Use descriptive names that match business terminology

### [Elements](https://likec4.dev/dsl/specification/#element-kind)

- Tags can be added to element kinds
- `title` - Keep it short
- `description` - Use descriptive titles that match business terminology
- `technology` - Be as specific as is helpful (`kotlin`, `Kafka`, `JSON + HTTP`, `gRPC`)
- `notation` - Helps identify logical element types, note the value is opaque to LikeC4, use the (?) in the bottom left corner of the rendered images to see this in action.
- `metadata`
    - **Source Documentation**: Always include links to source any information (wikis, file shares, repositories, etc. )
    - **Multiple Sources**: Include multiple source references where available, adopt a convention for element names, e.g. `source1`,`source2`,`wiki`,`slack`, etc.
    - **Descriptive Links**: Use descriptive text for links (e.g., "High Level Overview | URL") currently this is just decoration and renders "as-is"

#### [Styles](https://likec4.dev/dsl/styling/)
- Use a limited colour palette
- Use custom colours only if required, consider `col` prefix for custom colors to avoid confusion with real system elements
- Limit styles that have significant meaning (especially on a single view)
- Use limited icons for clarity, try to keep their style consistent
- Apply element shape to aid clarity
- Use size modifiers (xs, sm, lg) for visual hierarchy

### [Relationships](https://likec4.dev/dsl/relationships/)
- Be explicit when using relationships, e.g. (flows, uses, async, data)
- Apply style where it can clarify meaning
- Ensure it is clear which direction the relationship flows
- Keep the number of relationships small for the view being used, overly detailed relationship types can confuse more than help.

In this example we reverse the direction of the arrow for `subscribesTo` making it a more natural expression
```c4
specification {
  relationship calls {
    description 'a RPC between systems, calls to the dependency'
  }
  relationship subscribesTo {
    description 'receives information from a dependency'
    head none
    tail arrow
  }
}
```
In the model
```c4
models {
  queue newCustomers
  service any {
    any .subscribesTo newCustomers
  }
}
```

### [Tags](https://likec4.dev/dsl/specification/#tag)
- Use `UPPER_CASE`
- Prefixes can help but keep them short, e.g. for
    - domain (no prefix?)
    - technology (`TECH_`)
    - status (`STATUS_`)
- Steer clear of unusual initialisms or acronyms

### Example spec.c4

```c4
model {
  element actor {
    title 'Actor'
    notation 'Person'
    description 'Any human'
    style {
      color green
      shape person
    }
  }
  element api {
    description 'An HTTP API'
    technology 'JSON + HTTP'
    style {
      size xs
      icon https://www.svgrepo.com/show/93871/api.svg
    }
  }
}
```

Element types can be used in many ways including:
- usage (storage, queue, api)
- grouping (sub-domain, legacy)
- containers (service, external) for example to group items on a view

## 4. Models

Models define the actual architecture elements and their relationships.

```c4
domain marketing {
  #DOMAIN_MARKETING
  title 'Marketing Domain'
  description 'Marketing domain covers direct to customer interactions'
  api customer-preference {
    #DEPRECATED #LINE_OF_BUSINESS
    title 'Customer Marketing Preference'
    description 'All marketing preferences for an online customer'
    technology 'python'
    metadata {
      source1 'https://example.com/wiki'
      slack '#team-marketing | https://slack.com/ABC/DEF'
      email 'team@example.com'
    }

    customer-preference .uses salesforce.cloud {             
      description 'Uses Salesforce to store marketing preferences'
    }
  }
}
```

- Define relationships to dependencies inside an instance
  - Alternatives to the full name: `it`, `this`
- Use metadata for reference material
- Describe relationships for dependencies inside the source element to make them easier to find later

## 5. Views

### View Structure
Views define how models are projected and displayed.
- Put static views in the views folder
- Group related views in a single folder only if there are "too many"
- put dynamic views in `/views/use-cases`
- prefer query filters over explicit entity lists
- include the minimum possible entities to express the view

#### Global (predicates and styles)
Make use of the global block to DEFINE styles and predicates that can then be APPLIED later in views.
It can be useful to use `exclude` in predicate groups to remove what has already been pulled into a view via the view queries.

```c4
global {
  predicateGroup vNext {
    exclude *
      where tag is #LEGACY
  }
}
// apply
views {
  view system2 {
    global predicate vNext
    include marketing.*
  }
}
```

### View Best Practices

Remember this is a [C4 diagram](https://c4model.com/)
- Context
- Containers
- Components
- Classes (these never get drawn)

#### View Organization
- **Landscape Views**: Start with high-level landscape views showing all domains
- **Domain Views**: Create focused views for each business domain
- **Component Views**: Drill down into specific components when needed
- **Navigation**: Use `navigateTo` to link between related views

#### Predicate Groups
- **Business Domain Filtering**: Use predicates to filter by business domain (MARKETING, QUOTE)
- **Status Filtering**: Filter by status tags (TBD, TODO) for incomplete elements
- **Technology Filtering**: Filter by technology tags (HTTP, GRPC)
- **Functional Filtering**: Filter by functional tags (DOMAIN_UNDERWRITING, DOMAIN_PRICING)

#### Styling in Views
- **Global Styles**: Define styles that apply across all views
- **Element-Specific Styles**: Apply styles to specific element types or tags
- **Consistent Colors**: Use consistent color schemes across views
- **Visual Hierarchy**: Use size modifiers and shapes for visual clarity

#### View Naming and Documentation
- **Descriptive Titles**: Use clear, descriptive titles for each view
- **Descriptions**: Include descriptions explaining the view's purpose
- **Consistent Naming**: Use consistent naming patterns across views
- **Business Context**: Include business context in view descriptions

### View Patterns

#### Landscape View Pattern
- Include all top-level elements (`include *`)
- Use predicates to filter by business domain
- Include navigation hints to detailed views
- Provide business context in descriptions

#### Domain View Pattern
- Focus on specific domains (`view of someContainerModelElement`)
- Include all relevant elements within the domain (`include *`)
- Include related elements from other domains
- Use predicates to filter appropriately

#### Component View Pattern
- Drill down into specific components
- Show detailed relationships and dependencies
- Include technology information
- Focus on specific business flows

### View Organization Strategies

- Business Domain
- Technology
- Status
