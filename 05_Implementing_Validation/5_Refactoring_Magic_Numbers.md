# Refactoring Magic Numbers

Notice the first check below for the "customer.MembershipTypeId == 1"

No one will know what that means without looking at the database and checking what row corresponds to that value

This is called a **magic number** since it is hard to tell why it is there - we need to refactor these

```cs
namespace Vidly.Models
{
    public class Min18YearsIfAMember : ValidationAttribute
    {
        protected override ValidationResult IsValid(object value, ValidationContext validationContext)
        {
            var customer = (Customer) validationContext.ObjectInstance;

            if(customer.MembershipTypeId == 1)
                return ValidationResult.Success;

            if(customer.Birthdate == null)
                return new ValidationResult("Date of Birth is required");

            var age = DateTime.Today.Year - customer.Birthdate.Value.Year;

            return (age >= 18)
                ? ValidationResult.Success
                : new ValidationResult("Customer should be at least 18 years old to go on a membership");
        }
    }
}
```

We can create fields in our model to represent these "Magic Numbers" with names to represent their purpose

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;

namespace Vidly.Models
{
    public class MembershipType
    {
        public byte Id { get; set; }
        public short SignUpFee { get; set; }
        public byte DurationInMonths { get; set; }
        public byte DiscountRate { get; set; }
        public string Name { get; set; }

        public static readonly byte Unknown = 0;
        public static readonly byte PayAsYouGo = 1;
    }
}
```

Then we can include them in our code, and now it is easy to see what this code does

```cs
namespace Vidly.Models
{
    public class Min18YearsIfAMember : ValidationAttribute
    {
        protected override ValidationResult IsValid(object value, ValidationContext validationContext)
        {
            var customer = (Customer) validationContext.ObjectInstance;

            if(customer.MembershipTypeId == MembershipType.PayAsYouGo 
                || customer.MembershipTypeId == MembershipType.Unknown)
                return ValidationResult.Success;

            if(customer.Birthdate == null)
                return new ValidationResult("Date of Birth is required");

            var age = DateTime.Today.Year - customer.Birthdate.Value.Year;

            return (age >= 18)
                ? ValidationResult.Success
                : new ValidationResult("Customer should be at least 18 years old to go on a membership");
        }
    }
}
```
