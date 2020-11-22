# Adaptive Dialog DoWhile example

```cs
// Licensed under the MIT License.
// Copyright (c) Microsoft Corporation. All rights reserved.

using System;
using System.Runtime.CompilerServices;
using System.Threading;
using System.Threading.Tasks;
using AdaptiveExpressions.Properties;
using Newtonsoft.Json;

namespace Microsoft.Bot.Builder.Dialogs.Adaptive.Actions
{
    /// <summary>
    /// Executes a set of actions while an expression evaluates to true.
    /// </summary>
    public class DoWhile : ActionScope
    {
        /// <summary>
        /// Class identifier.
        /// </summary>
        [JsonProperty("$kind")]
        public const string Kind = "Microsoft.DoWhile";

        /// <summary>
        /// Initializes a new instance of the <see cref="DoWhile"/> class.
        /// </summary>
        /// <param name="sourceFilePath">Optional, full path of the source file that contains the caller.</param>
        /// <param name="sourceLineNumber">optional, line number in the source file at which the method is called.</param>
        [JsonConstructor]
        public DoWhile([CallerFilePath] string sourceFilePath = "", [CallerLineNumber] int sourceLineNumber = 0)
            : base()
        {
            this.RegisterSourceLocation(sourceFilePath, sourceLineNumber);
        }

        /// <summary>
        /// Gets or sets an optional expression which if is true will disable this action.
        /// </summary>
        /// <example>
        /// "user.age > 18".
        /// </example>
        /// <value>
        /// A boolean expression. 
        /// </value>
        [JsonProperty("disabled")]
        public BoolExpression Disabled { get; set; } 

        /// <summary>
        /// Gets or sets the memory expression evaludating to a condition which determines if the DoWhile should continue.
        /// </summary>
        /// <example>
        /// "user.age > 18".
        /// </example>
        /// <value>
        /// The memory expression which determines if the DoWhile should continue.
        /// </value>
        [JsonProperty("condition")]
        public BoolExpression Condition { get; set; } = false;

        /// <summary>
        /// Called when the dialog is started and pushed onto the dialog stack.
        /// </summary>
        /// <param name="dc">The <see cref="DialogContext"/> for the current turn of conversation.</param>
        /// <param name="options">Optional, initial information to pass to the dialog.</param>
        /// <param name="cancellationToken">A cancellation token that can be used by other objects
        /// or threads to receive notice of cancellation.</param>
        /// <returns>A <see cref="Task"/> representing the asynchronous operation.</returns>
        public override async Task<DialogTurnResult> BeginDialogAsync(DialogContext dc, object options = null, CancellationToken cancellationToken = default(CancellationToken))
        {
            if (options is CancellationToken)
            {
                throw new ArgumentException($"{nameof(options)} cannot be a cancellation token");
            }

            if (this.Disabled != null && this.Disabled.GetValue(dc.State) == true)
            {
                return await dc.EndDialogAsync(cancellationToken: cancellationToken).ConfigureAwait(false);
            }

            return await base.BeginDialogAsync(dc, options, cancellationToken).ConfigureAwait(false);
        }

        /// <summary>
        /// Called when the dialog's action ends.
        /// </summary>
        /// <param name="dc">The <see cref="DialogContext"/> for the current turn of conversation.</param>
        /// <param name="result">Optional, value returned from the dialog that was called. The type
        /// of the value returned is dependent on the child dialog.</param>
        /// <param name="cancellationToken">A cancellation token that can be used by other objects
        /// or threads to receive notice of cancellation.</param>
        /// <returns>A <see cref="Task"/> representing the asynchronous operation.</returns>
        protected override async Task<DialogTurnResult> OnEndOfActionsAsync(DialogContext dc, object result = null, CancellationToken cancellationToken = default)
        {
            if (Condition.GetValue(dc.State))
            {
                return await this.BeginActionAsync(dc, 0, cancellationToken).ConfigureAwait(false);
            }

            return await base.OnEndOfActionsAsync(dc, result, cancellationToken).ConfigureAwait(false);
        }

        /// <summary>
        /// Builds the compute Id for the dialog.
        /// </summary>
        /// <returns>A string representing the compute Id.</returns>
        protected override string OnComputeId()
        {
            return $"{this.GetType().Name}({this.Condition?.ToString()})";
        }
    }
}

```

```json
{
    "$schema": "https://schemas.botframework.com/schemas/component/v1.0/component.schema",
    "$role": "implements(Microsoft.IDialog)",
    "title": "Do While Loop",
    "description": "Execute actions while a condition is true.",
    "type": "object",
    "properties": {
        "id": {
            "type": "string",
            "title": "Id",
            "description": "Optional id for the dialog"
        },
        "disabled": {
            "$ref": "schema:#/definitions/booleanExpression",
            "title": "Disabled",
            "description": "Optional condition which if true will disable this action.",
            "examples": [
                "user.age > 18"
            ]
        },
        "condition": {
            "$ref": "schema:#/definitions/booleanExpression",
            "title": "Disabled",
            "description": "Optional condition which must evaluate to true for actions to continue processing.",
            "examples": [
                "user.age > 18"
            ]
        },
        "actions": {
            "type": "array",
            "items": {
                "$kind": "Microsoft.IDialog"
            },
            "title": "Actions",
            "description": "Actions to execute while the condition is true."
        }
    }
}
```

Example in Composer:



