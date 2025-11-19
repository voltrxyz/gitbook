# Core Components Implementation

## Core Components Guide

This guide details the essential components and instructions required to implement a Voltr adaptor. Each adaptor must implement these core components to maintain compatibility with the Voltr vault system.

### Required Instructions

Every adaptor must implement these three core instructions:

#### 1. Initialize Instruction

```rust
pub struct Initialize<'info> {
    /// The payer for account creation
    #[account(mut)]
    pub payer: Signer<'info>,

    /// Vault's strategy authority
    #[account(mut)]
    pub vault_strategy_auth: Signer<'info>,

    /// The strategy account
    #[account(
        seeds = [constants::STRATEGY_SEED, strategy.load()?.counterparty_asset_ta.key().as_ref()],
        bump = strategy.load()?.bump
    )]
    pub strategy: AccountLoader<'info, Strategy>,

    pub system_program: Program<'info, System>,

    /// The protocol's program ID
    /// CHECK: Validated in handler
    #[account(constraint = protocol_program.key() == strategy.load()?.protocol_program)]
    pub protocol_program: AccountInfo<'info>,
}
```

Key responsibilities:

* Create protocol-specific accounts
* Initialize state tracking
* Setup token accounts if needed
* Configure protocol-specific parameters

#### 2. Deposit Instruction

```rust
pub struct Deposit<'info> {
    /// Strategy authority
    #[account(mut)]
    pub vault_strategy_auth: Signer<'info>,

    /// Strategy account
    #[account(
        seeds = [constants::STRATEGY_SEED, counterparty_asset_ta.key().as_ref()],
        bump = strategy.load()?.bump
    )]
    pub strategy: AccountLoader<'info, Strategy>,

    /// Asset mint
    #[account(mut)]
    pub vault_asset_mint: Box<InterfaceAccount<'info, Mint>>,

    /// Strategy's asset ATA
    #[account(
        mut,
        associated_token::mint = vault_asset_mint,
        associated_token::authority = vault_strategy_auth,
        associated_token::token_program = asset_token_program,
    )]
    pub vault_strategy_asset_ata: Box<InterfaceAccount<'info, TokenAccount>>,

    pub asset_token_program: Interface<'info, TokenInterface>,

    /// Protocol's token account
    #[account(mut)]
    pub counterparty_asset_ta: Box<InterfaceAccount<'info, TokenAccount>>,

    /// Protocol program
    /// CHECK: Validated in handler
    #[account(constraint = protocol_program.key() == strategy.load()?.protocol_program)]
    pub protocol_program: AccountInfo<'info>,
}
```

Required functionality:

* Validate deposit amount
* Handle token transfers
* Interact with protocol
* Track position value
* Return final position amount

#### 3. Withdraw Instruction

```rust
pub struct Withdraw<'info> {
    /// Strategy authority
    #[account(mut)]
    pub vault_strategy_auth: Signer<'info>,

    /// Strategy account
    #[account(
        seeds = [constants::STRATEGY_SEED, counterparty_asset_ta.key().as_ref()],
        bump = strategy.load()?.bump
    )]
    pub strategy: AccountLoader<'info, Strategy>,

    /// Asset mint
    #[account(mut)]
    pub vault_asset_mint: Box<InterfaceAccount<'info, Mint>>,

    /// Strategy's asset ATA
    #[account(
        mut,
        associated_token::mint = vault_asset_mint,
        associated_token::authority = vault_strategy_auth,
        associated_token::token_program = asset_token_program,
    )]
    pub vault_strategy_asset_ata: Box<InterfaceAccount<'info, TokenAccount>>,

    pub asset_token_program: Interface<'info, TokenInterface>,

    /// CHECK: Validated in handler
    #[account(mut)]
    pub counterparty_asset_ta_auth: AccountInfo<'info>,

    #[account(mut)]
    pub counterparty_asset_ta: Box<InterfaceAccount<'info, TokenAccount>>,

    /// CHECK: Validated in handler
    #[account(constraint = protocol_program.key() == strategy.load()?.protocol_program)]
    pub protocol_program: AccountInfo<'info>,
}
```

Required functionality:

* Validate withdrawal amount
* Handle protocol withdrawal
* Transfer tokens back to vault
* Update position tracking
* Return final position amount

### State Management

#### Strategy Account

```rust
#[account(zero_copy(unsafe))]
#[repr(C)]
pub struct Strategy {
    /// Strategy name (fixed-length)
    pub name: [u8; 32],

    /// Strategy description (fixed-length)
    pub description: [u8; 64],

    /// Protocol's token account
    pub counterparty_asset_ta: Pubkey,

    /// Protocol program ID
    pub protocol_program: Pubkey,

    /// Version for migrations
    pub version: u8,

    /// PDA bump
    pub bump: u8,

    /// Strategy type
    pub strategy_type: StrategyType,

    /// Reserved space
    pub _reserved: [u8; 128],
}
```

#### Strategy Type Definition

```rust
#[derive(AnchorSerialize, AnchorDeserialize, Clone, Copy, Debug, PartialEq, Eq)]
#[repr(u8)]
pub enum StrategyType {
    Solend,
    Drift,
    Marginfi,
    Kamino,
    // Add your protocol type here
}
```

### Position Value Calculation

Your adaptor must implement accurate position value calculations:

```rust
fn calculate_current_amount(
    collateral_amount: u64,
    reserve_info: &AccountInfo
) -> Result<u64> {
    // Protocol-specific calculation logic
    // Must handle:
    // 1. Token decimals
    // 2. Exchange rates
    // 3. Accrued interest/rewards
    // 4. Protocol-specific factors
}
```

### Error Handling

Implement comprehensive error handling:

```rust
#[error_code]
pub enum AdaptorError {
    #[msg("Invalid amount provided.")]
    InvalidAmount,
    
    #[msg("Invalid account owner.")]
    InvalidAccountOwner,
    
    #[msg("Already initialized.")]
    AlreadyInitialized,
    
    #[msg("Invalid token mint.")]
    InvalidTokenMint,
    
    #[msg("Invalid account input.")]
    InvalidAccountInput,
    
    #[msg("Invalid reserve account.")]
    InvalidReserveAccount,
    
    #[msg("Not rent exempt.")]
    NotRentExempt,
    
    #[msg("Math overflow.")]
    MathOverflow,
    
    // Add protocol-specific errors
}
```

### Account Validation Patterns

#### Token Account Validation

```rust
fn validate_token_accounts(
    vault_asset_mint: &Pubkey,
    strategy_ata: &Pubkey,
    vault_strategy_auth: &Pubkey
) -> Result<()> {
    // 1. Check mint matches
    require!(
        reserve_liquidity_mint == vault_asset_mint.key(),
        AdaptorError::InvalidReserveAccount
    );

    // 2. Verify ATA derivation
    require!(
        get_associated_token_address_with_program_id(
            vault_strategy_auth,
            vault_asset_mint,
            &TOKEN_PROGRAM_ID
        ) == *strategy_ata,
        AdaptorError::InvalidAccountInput
    );

    Ok(())
}
```

#### Protocol Account Validation

```rust
fn validate_protocol_accounts(
    program_id: &Pubkey,
    accounts: &[AccountInfo]
) -> Result<()> {
    // 1. Check program ownership
    require!(
        protocol_program.key() == strategy.load()?.protocol_program,
        AdaptorError::InvalidProtocolProgram
    );

    // 2. Verify account ownership
    require!(
        account.owner == program_id,
        AdaptorError::InvalidAccountOwner
    );

    // 3. Protocol-specific validation
    
    Ok(())
}
```

### Implementation Checklist

Before deploying your adaptor, ensure you have:

* [ ] Implemented all required instructions
* [ ] Added proper account validation
* [ ] Implemented accurate position tracking
* [ ] Added comprehensive error handling
* [ ] Handled token decimals correctly
* [ ] Added protocol-specific security checks
* [ ] Tested all error cases
* [ ] Validated token account handling
* [ ] Added protocol-specific cleanup logic

### Security Requirements

1. **Account Validation**
   * Always validate account ownership
   * Check PDA derivation
   * Verify token account authorities
   * Validate protocol accounts
2. **Amount Validation**
   * Use checked math operations
   * Handle decimal conversions safely
   * Validate against protocol limits
   * Check for overflows
3. **State Updates**
   * Update position values atomically
   * Maintain consistent state
   * Handle failed transactions
   * Track version numbers
