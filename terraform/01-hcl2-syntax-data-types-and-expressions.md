# HashiCorp Configuration Language (HCL2): Syntax, Types & Expressions

---

## 1. HCL2 Data Types & Structures

| Data Type | Description & Syntax | Example |
| :--- | :--- | :--- |
| **`string`** | Unicode text sequence | `"10.10.10.0/24"` |
| **`number`** | Integer or floating-point value | `502`, `1.5` |
| **`bool`** | Boolean flag | `true`, `false` |
| **`list(type)`** | Ordered indexed sequence of elements | `["10.10.10.0/24", "10.10.20.0/24"]` |
| **`set(type)`** | Unordered collection of unique elements | `toset(["us-east-1a", "us-east-1b"])` |
| **`map(type)`** | Collection of key-value pairs (same type) | `{ env = "prod", tier = "data" }` |
| **`object({...})`**| Complex structure with named attributes of differing types | `object({ name = string, port = number, enabled = bool })` |
| **`tuple([...])`** | Fixed-length sequence of differing types | `tuple([string, number, bool])` |

---

## 2. Iteration: `count` vs `for_each`

* **`count` (Index-based):**
  * Uses integer index `count.index`.
  * *Disadvantage:* Removing an item from the middle of a list shifts indices and forces unnecessary recreation of downstream resources.
* **`for_each` (Key-based - Best Practice):**
  * Iterates over a map or set of strings using `each.key` and `each.value`.
  * *Advantage:* Resources are tracked by distinct keys (e.g. `aws_subnet.private["app-1a"]`), allowing isolated additions or deletions without modifying other resources.

```hcl
# for_each with Map of Objects Example
variable "subnets" {
  type = map(object({
    cidr = string
    az   = string
  }))
  default = {
    "dmz-1a"  = { cidr = "10.10.20.0/24", az = "us-east-1a" }
    "corp-1a" = { cidr = "10.10.10.0/24", az = "us-east-1a" }
    "ot-1a"   = { cidr = "10.10.30.0/24", az = "us-east-1a" }
  }
}

resource "aws_subnet" "managed" {
  for_each          = var.subnets
  vpc_id            = aws_vpc.main.id
  cidr_block        = each.value.cidr
  availability_zone = each.value.az
  tags = {
    Name = each.key
  }
}
```

---

## 3. Dynamic Blocks & Conditional Expressions

```hcl
# Dynamic Ingress Rules in Security Group
resource "aws_security_group" "firewall_ports" {
  name   = "soc-perimeter-sg"
  vpc_id = aws_vpc.main.id

  dynamic "ingress" {
    for_each = var.allowed_ingress_rules
    content {
      description = ingress.value.desc
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = [ingress.value.cidr]
    }
  }
}

# Ternary Conditional Expression
locals {
  instance_count = var.environment == "production" ? 3 : 1
}
```
