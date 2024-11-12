# D3 Solana Contract Technical Documentation

## Table of Contents
1. [Context Definitions](#context-definitions)
   - [Administrative Setup](#administrative-setup)
   - [Payment and Registration](#payment-and-registration)
   - [Minter Authority](#minter-authority)
   - [Administrative Control](#administrative-control)
2. [Program Derived Addresses (PDAs)](#program-derived-addresses)
3. [Core Functions](#core-functions)
4. [Helper Functions](#helper-functions)
5. [Events and Vouchers](#events-and-vouchers)

## Context Definitions

### Administrative Setup

#### Initialize
```rust
pub struct Initialize<'info> {
    pub config: Account<'info, Config>,
    pub authority: Signer<'info>,
    pub collection: Signer<'info>,
    pub mpl_core_program: UncheckedAccount<'info>,
    pub system_program: Program<'info, System>,
}
```
**Authority:** Initial deployer  
**Purpose:** Protocol initialization and configuration  
**Usage:** One-time initialization of protocol configuration and collection setup

#### AddTld
```rust
pub struct AddTld<'info> {
    pub config: Account<'info, Config>,
    pub tld_entry: Account<'info, TldEntry>,
    pub authority: Signer<'info>,
    pub system_program: Program<'info, System>,
}
```
**Authority:** Protocol authority  
**Purpose:** Register new top-level domains in the protocol  
**Usage:** Adding new TLDs to the system for domain registration

#### RemoveTld
```rust
pub struct RemoveTld<'info> {
    pub config: Account<'info, Config>,
    pub tld_entry: Account<'info, TldEntry>,
    pub authority: Signer<'info>,
    pub system_program: Program<'info, System>,
}
```
**Authority:** Protocol authority  
**Purpose:** Deactivate existing top-level domains  
**Usage:** Removing or deactivating TLDs from the system

### Payment and Registration

#### Pay
```rust
pub struct Pay<'info> {
    pub config: Box<Account<'info, Config>>,
    pub buyer: Signer<'info>,
    pub treasury_account: Option<AccountInfo<'info>>,
    pub treasury_token_account: Option<Account<'info, TokenAccount>>,
    pub buyer_token_account: Option<Account<'info, TokenAccount>>,
    pub collection: Account<'info, BaseCollectionV1>,
    pub asset: Signer<'info>,
    pub used_payment_id: Box<Account<'info, UsedPaymentIds>>,
    pub domain: Account<'info, Domain>,
    pub tld_entry: Box<Account<'info, TldEntry>>,
    pub authority: AccountInfo<'info>,
    pub token_program: Program<'info, Token>,
    pub system_program: Program<'info, System>,
    pub mpl_core_program: UncheckedAccount<'info>,
    pub ix_sysvar: AccountInfo<'info>,
}
```
**Authority:** Any user with valid payment voucher  
**Purpose:** Process primary domain registration and payment  
**Usage:** Primary sale registration of new domains with payment processing

#### RenewAssetUser
```rust
pub struct RenewAssetUser<'info> {
    pub config: Box<Account<'info, Config>>,
    pub buyer: Signer<'info>,
    pub treasury_account: Option<AccountInfo<'info>>,
    pub treasury_token_account: Option<Account<'info, TokenAccount>>,
    pub buyer_token_account: Option<Account<'info, TokenAccount>>,
    pub domain: Box<Account<'info, Domain>>,
    pub asset: Account<'info, BaseAssetV1>,
    pub used_payment_id: Box<Account<'info, UsedPaymentIds>>,
    pub token_program: Program<'info, Token>,
    pub system_program: Program<'info, System>,
    pub mpl_core_program: UncheckedAccount<'info>,
    pub ix_sysvar: AccountInfo<'info>,
}
```
**Authority:** Domain owner  
**Purpose:** Process domain renewal by current owner  
**Usage:** Renewal of existing domains by their current owners

#### SecondarySale
```rust
pub struct SecondarySale<'info> {
    pub config: Account<'info, Config>,
    pub buyer: Signer<'info>,
    pub treasury: AccountInfo<'info>,
    pub used_payment_id: Account<'info, UsedPaymentIds>,
    pub collection: Account<'info, BaseCollectionV1>,
    pub seller_account: AccountInfo<'info>,
    pub asset: Account<'info, BaseAssetV1>,
    pub system_program: Program<'info, System>,
    pub mpl_core_program: UncheckedAccount<'info>,
    pub ix_sysvar: AccountInfo<'info>,
}
```
**Authority:** Any user with valid secondary sale voucher  
**Purpose:** Process secondary market domain transfers  
**Usage:** Facilitates domain transfers in secondary market transactions

### Minter Authority Contexts

#### OrderMint
```rust
pub struct OrderMint<'info> {
    pub config: Account<'info, Config>,
    pub asset: Signer<'info>,
    pub collection: Account<'info, BaseCollectionV1>,
    pub authority: AccountInfo<'info>,
    pub to: AccountInfo<'info>,
    pub minter: Signer<'info>,
    pub tld: Account<'info, TldEntry>,
    pub domain: Account<'info, Domain>,
    pub mpl_core_program: UncheckedAccount<'info>,
    pub system_program: Program<'info, System>,
}
```
**Authority:** Authorized minter  
**Purpose:** Direct domain minting for orders  
**Usage:** Minting domains through authorized minter account  
**Validation:**
- Minter must be authorized in config
- TLD must be active
- Domain must not exist

#### Renew
```rust
pub struct Renew<'info> {
    pub config: Account<'info, Config>,
    pub asset: Account<'info, BaseAssetV1>,
    pub minter: Signer<'info>,
    pub domain: Account<'info, Domain>,
    pub system_program: Program<'info, System>,
}
```
**Authority:** Authorized minter  
**Purpose:** Process domain renewal by minter  
**Usage:** Renewing domains through authorized minter account  
**Validation:**
- Minter must be authorized
- Domain must exist
- Must be within renewal period

### Administrative Control Contexts

#### SetUri
```rust
pub struct SetUri<'info> {
    pub config: Account<'info, Config>,
    pub collection: Account<'info, BaseCollectionV1>,
    pub authority: Signer<'info>,
    pub system_program: Program<'info, System>,
    pub mpl_core_program: UncheckedAccount<'info>,
}
```
**Authority:** Protocol authority  
**Purpose:** Update collection metadata URI  
**Usage:** Updating metadata URI for the entire collection  
**Validation:**
- Must be protocol authority
- Collection must exist

#### SetGracePeriod
```rust
pub struct SetGracePeriod<'info> {
    pub config: Account<'info, Config>,
    pub authority: Signer<'info>,
    pub system_program: Program<'info, System>,
}
```
**Authority:** Protocol authority  
**Purpose:** Modify domain renewal grace period  
**Usage:** Adjusting the grace period for domain renewals  
**Validation:**
- Must be protocol authority
- New grace period must be valid

#### BurnAsset
```rust
pub struct BurnAsset<'info> {
    pub config: Account<'info, Config>,
    pub asset: Account<'info, BaseAssetV1>,
    pub collection: Account<'info, BaseCollectionV1>,
    pub authority: Signer<'info>,
    pub domain: Account<'info, Domain>,
    pub mpl_core_program: UncheckedAccount<'info>,
    pub system_program: Program<'info, System>,
}
```
**Authority:** Protocol authority  
**Purpose:** Administrative domain burning  
**Usage:** Forcefully burning domains in special circumstances  
**Validation:**
- Must be protocol authority
- Domain must exist
- Asset must be valid

#### SetBlockAllTransfers
```rust
pub struct SetBlockAllTransfers<'info> {
    pub config: Account<'info, Config>,
    pub authority: Signer<'info>,
    pub collection: Account<'info, BaseCollectionV1>,
    pub mpl_core_program: UncheckedAccount<'info>,
    pub system_program: Program<'info, System>,
}
```
**Authority:** Protocol authority  
**Purpose:** Enable/disable global transfer restrictions  
**Usage:** Emergency stop for all domain transfers  
**Validation:**
- Must be protocol authority
- Affects entire collection

#### AdminSetLockStatus
```rust
pub struct AdminSetLockStatus<'info> {
    pub config: Account<'info, Config>,
    pub authority: Signer<'info>,
    pub transfer_lock: Account<'info, TransferLock>,
    pub collection: Account<'info, BaseCollectionV1>,
    pub asset: Account<'info, BaseAssetV1>,
    pub mpl_core_program: UncheckedAccount<'info>,
    pub system_program: Program<'info, System>,
}
```
**Authority:** Protocol authority  
**Purpose:** Set transfer lock status for specific domain  
**Usage:** Granular control over individual domain transfers  
**Validation:**
- Must be protocol authority
- Domain must exist
- Transfer lock must be initialized

#### AdminSafeTransfer
```rust
pub struct AdminSafeTransfer<'info> {
    pub config: Account<'info, Config>,
    pub transfer_lock: Account<'info, TransferLock>,
    pub asset: Account<'info, BaseAssetV1>,
    pub collection: Account<'info, BaseCollectionV1>,
    pub authority: Signer<'info>,
    pub to: AccountInfo<'info>,
    pub mpl_core_program: UncheckedAccount<'info>,
    pub system_program: Program<'info, System>,
}
```
**Authority:** Protocol authority  
**Purpose:** Administrative override for domain transfers  
**Usage:** Force transfer domains in special circumstances  
**Validation:**
- Must be protocol authority
- Domain must exist
- Transfer lock status must be verified
- Target account must be valid

## Program Derived Addresses (PDAs)

### 1. Config PDA

```rust
Seeds: ["config"]
Account Type: Config

pub struct Config {
    pub voucher_signer: Pubkey,
    pub authority: Pubkey,
    pub minter: Pubkey,
    pub treasury: Pubkey,
    pub secondary_fee: u16,
    pub grace_period: i64,
    pub pending_delete_period: i64,
    pub block_all_transfers: bool,
}
```

**Purpose:**
- Central configuration storage for protocol
- Management of protocol-wide parameters
- Storage of critical protocol addresses

**Access Control:**
- Only authority can modify
- Anyone can read
- No direct modifications allowed

**Used In:**
- All administrative operations
- Payment processing
- Domain management
- Transfer controls

### 2. Domain PDA

```rust
Seeds: ["domain", label.as_bytes(), tld.as_bytes()]
Account Type: Domain

pub struct Domain {
    pub asset: Pubkey,
    pub expiration: i64,
    pub is_locked: bool,
}
```

**Purpose:**
- Stores individual domain registration details
- Links domains to NFT assets
- Manages domain status and expiration

**Access Control:**
- Minter can create
- Authority can modify lock status
- Anyone can read

**Used In:**
- Domain registration
- Renewal operations
- Transfer operations
- Status checks

### 3. TLD Entry PDA

```rust
Seeds: ["tld_entry", tld.as_bytes()]
Account Type: TldEntry

pub struct TldEntry {
    pub name: String,
    pub active: bool,
}
```

**Purpose:**
- Manages available top-level domains
- Controls TLD activation status
- Prevents duplicate TLDs

**Access Control:**
- Only authority can add/remove
- Anyone can read
- Must be active for domain registration

**Used In:**
- TLD registration
- Domain registration validation
- TLD management
- Status checks

### 4. Used Payment IDs PDA

```rust
Seeds: ["used_payment_id", payment_id.as_bytes()]
Account Type: UsedPaymentIds

pub struct UsedPaymentIds {
    // Empty struct - existence indicates used payment ID
}
```

**Purpose:**
- Prevents payment replay attacks
- Ensures payment uniqueness
- Transaction idempotency control

**Access Control:**
- Created during payment processing
- Read-only after creation
- Never modified or deleted

**Used In:**
- Primary sale payments
- Secondary sale payments
- Renewal payments

### 5. Transfer Lock PDA

```rust
Seeds: ["transfer_lock", asset.key().as_ref()]
Account Type: TransferLock

pub struct TransferLock {
    pub locked: bool,
}
```

**Purpose:**
- Individual domain transfer control
- Granular transfer restrictions
- Compliance management

**Access Control:**
- Authority can set lock status
- Anyone can read
- Checked during transfers

**Used In:**
- Administrative lock settings
- Transfer validations
- Safe transfer operations

## Core Functions

### 1. Mint Implementation

```rust
pub fn mint<'info>(
    asset: &mut Signer<'info>,
    collection: &AccountInfo<'info>,
    domain: &mut Account<Domain>,
    authority: &AccountInfo<'info>,
    payer: &mut Signer<'info>,
    target_owner: &AccountInfo<'info>,
    mpl_core_program: &AccountInfo<'info>,
    system_program: &AccountInfo<'info>,
    tld_entry: &mut Account<TldEntry>,
    config_address: Pubkey,
    label: String,
    expiration_timestamp: i64,
    grace_period: i64,
    token_uri: String,
    block_transfers: bool,
) -> Result<()>
```

#### Purpose:
- Creates new domain NFT registration
- Sets up domain metadata and ownership
- Establishes expiration and transfer settings

#### Validation Steps
1. **Parameter Validation**
```rust
fn validate_mint_parameters(
    label: &str,
    expiration: i64,
    block_transfers: bool,
    grace_period: i64,
) -> Result<()>
```
- Label length check (1-63 characters)
- Expiration time validation
- Transfer settings verification
- Grace period boundaries

2. **Domain Initialization**
```rust
fn initialize_domain(
    domain: &mut Account<Domain>,
    asset_key: Pubkey,
    expiration: i64,
) -> Result<()>
```
- Set asset public key
- Initialize expiration time
- Set initial lock status

3. **NFT Creation**
```rust
fn mint_domain_token(
    // Parameters
) -> Result<()>
```
- Create asset plugins
- Set up metadata
- Configure ownership

#### Error Cases
```rust
pub enum D3Error {
    #[msg("Invalid label length")]
    InvalidLabelLength,
    
    #[msg("Invalid domain expiration")]
    InvalidDomainExpiration,
    
    #[msg("All transfers are blocked")]
    AllTransfersAreBlocked,
    
    #[msg("Minted and expired outside grace period")]
    MintedAndExpiredOutsideGracePeriod,
}
```

#### Events Emitted
```rust
emit!(SLDMinted {
    asset: asset.key(),
    to: *target_owner.key,
    label: label.clone(),
    tld: tld_entry.name.clone(),
    expiration: expiration_timestamp,
});
```

### 2. Renew Implementation

```rust
pub fn renew_domain<'info>(
    domain: &mut Account<Domain>,
    asset: &Account<'info, BaseAssetV1>,
    new_expiration_timestamp: i64,
    grace_period: i64,
    order_id: String,
) -> Result<()>
```

#### Purpose:
- Extends domain registration period
- Updates domain expiration timestamp
- Maintains domain ownership and metadata

#### Component Functions

1. **Expiration Validation**
```rust
fn validate_expiration_time(
    expiration: i64
) -> Result<()>
```

2. **Ownership Validation**
```rust
fn validate_asset_ownership(
    asset: &Account<BaseAssetV1>
) -> Result<()>
```

3. **Renewal Period Validation**
```rust
fn validate_renewal_period(
    current_expiration: i64,
    grace_period: i64
) -> Result<()>
```

4. **Domain Update**
```rust
fn update_domain_expiration(
    domain: &mut Account<Domain>,
    asset: &Account<BaseAssetV1>,
    new_expiration: i64,
    order_id: String,
) -> Result<()>
```

#### Error Cases
```rust
pub enum D3Error {
    #[msg("Invalid domain expiration")]
    InvalidDomainExpiration,
    
    #[msg("Invalid asset owner")]
    InvalidAssetOwner,
    
    #[msg("Renewal outside grace period")]
    RenewalOutsideGracePeriod,
    
    #[msg("Too big expiration time")]
    TooBigExpirationTime,
}
```

#### Events Emitted
```rust
emit!(SLDRenewed {
    asset: asset.key(),
    expiration: new_expiration,
    order_id,
});
```
## Helper Functions

### 1. Ed25519 Signature Verification Helpers

#### High-Level Signature Verification
```rust
pub fn verify_signature(
    config: &Config,
    voucher: &impl VoucherSignature,
    signature: [u8; 64],
    ix_sysvar: &AccountInfo,
) -> Result<()>
```
**Purpose:**
- Validates payment voucher signatures
- Ensures transaction authenticity
- Prevents unauthorized operations

**Implementation Flow:**
1. Get signature data from voucher
2. Load instruction for verification
3. Verify using Ed25519 program
4. Handle verification results


### 2. Token Transfer Helpers

#### Native SOL Transfer
```rust
pub fn transfer_sol<'info>(
    from: &AccountInfo<'info>,
    to: &AccountInfo<'info>,
    system_program: &AccountInfo<'info>,
    amount: u64,
) -> Result<()>
```
**Security Checks:**
- Balance verification before transfer
- Account validation
- Proper instruction invocation

#### SPL Token Transfer
```rust
pub fn transfer_tokens<'info>(
    from_account: &Account<'info, TokenAccount>,
    to_account: &Account<'info, TokenAccount>,
    authority: &AccountInfo<'info>,
    token_program: &AccountInfo<'info>,
    amount: u64,
) -> Result<()>
```
**Validation Steps:**
- Amount validation
- Balance verification
- Account ownership checks
- Token program verification

### Common Error Cases

```rust
pub enum D3Error {
    #[msg("Signature verification failed.")]
    SigVerificationFailed,
    
    #[msg("Insufficient funds")]
    InsufficientFunds,
    
    #[msg("Invalid payment amount")]
    InvalidPaymentAmount,
    
    #[msg("Transfer failed")]
    TransferFailed,
}
```

## Events and Payment Vouchers

### System Events

#### 1. Payment Events

##### PaymentFulfilled
```rust
#[event]
pub struct PaymentFulfilled {
    pub payment_id: String,
    pub buyer: Pubkey,
    pub amount: u64,
    pub token: Pubkey,
    pub immediate_mint: bool,
}
```
**Purpose:**
- Track successful payment completion
- Record payment details for indexing
- Monitor immediate minting status

**Emission Points:**
- Primary sale completion
- Secondary sale completion
- Domain renewal processing
- Payment verification success

#### 2. Domain Events

##### SLDMintedForOrder
```rust
#[event]
pub struct SLDMintedForOrder {
    pub asset: Pubkey,
    pub to: Pubkey,
    pub label: String,
    pub tld: String,
    pub expiration: i64,
    pub order_id: String,
}
```
**Purpose:**
- Record domain minting with order tracking
- Track initial domain ownership
- Maintain registration history

**Emission Points:**
- OrderMint instruction execution
- Pay instruction with immediate mint
- Administrative minting operations

##### SLDMinted
```rust
#[event]
pub struct SLDMinted {
    pub asset: Pubkey,
    pub to: Pubkey,
    pub label: String,
    pub tld: String,
    pub expiration: i64,
}
```
**Purpose:**
- Record direct domain minting
- Track domain creation without orders
- Maintain domain registration records

**Emission Points:**
- Direct minting operations
- Administrative minting
- System-initiated minting

##### SLDRenewed
```rust
#[event]
pub struct SLDRenewed {
    pub asset: Pubkey,
    pub expiration: i64,
    pub order_id: String,
}
```
**Purpose:**
- Track domain renewal operations
- Record expiration updates
- Maintain renewal history

**Emission Points:**
- Domain renewal completion
- Administrative renewal
- Batch renewal operations

#### 3. Administrative Events

##### LockStatusChanged
```rust
#[event]
pub struct LockStatusChanged {
    pub asset_address: Pubkey,
    pub is_transfer_locked: bool,
}
```
**Purpose:**
- Monitor transfer lock changes
- Track administrative actions
- Maintain compliance records

**Emission Points:**
- AdminSetLockStatus execution
- Emergency lock operations
- Administrative overrides

##### TLDRemoved
```rust
#[event]
pub struct TLDRemoved {
    pub tld: String,
}
```
**Purpose:**
- Track TLD deactivation
- Monitor TLD management
- Maintain TLD history

**Emission Points:**
- RemoveTld instruction execution
- TLD deactivation operations
- Administrative TLD management

### Payment Voucher Structures

#### 1. Primary Sale Voucher
```rust
pub struct PaymentVoucher {
    pub buyer: Pubkey,             // Authorized buyer address
    pub token: Pubkey,             // Payment token (NATIVE or SPL)
    pub amount: u64,               // Total payment amount
    pub voucher_expiration: i64,   // Voucher validity timestamp
    pub payment_id: String,        // Unique payment identifier
    pub order_id: String,          // Order tracking ID
    pub names: Vec<NameMintInfo>,  // Domains to process
}

pub struct NameMintInfo {
    pub registry: Pubkey,          // Registry contract address
    pub label: String,             // Domain name
    pub tld: String,               // Top-level domain
    pub expiration_time: u64,      // Domain expiration
    pub owner: Pubkey,            // Target owner
    pub renewal: bool,            // Is renewal operation
}
```

**Validation Requirements:**
1. Signature Verification
   - Valid buyer signature
   - Non-expired voucher
   - Authorized signer

2. Payment Validation
   - Correct amount calculation
   - Valid token address
   - Sufficient funds

3. Domain Validation
   - Valid label format
   - Active TLD
   - Available domain name

#### 2. Secondary Sale Voucher
```rust
pub struct SecondarySaleVoucher {
    pub buyer: Pubkey,             // Authorized buyer address
    pub amount: u64,               // Total payment amount
    pub voucher_expiration: i64,   // Voucher validity timestamp
    pub payment_id: String,        // Unique payment identifier
    pub names: Vec<NameTransferInfo>, // Domains to transfer
}

pub struct NameTransferInfo {
    pub registry: Pubkey,          // Registry contract address
    pub tokenId: u256,            // Token identifier
    pub price: u64,               // Individual token price
}
```

**Validation Requirements:**
1. Transfer Validation
   - Valid token ownership
   - Transfer permission
   - Unlock status

2. Payment Validation
   - Price verification
   - Fee calculation
   - Total amount matching

3. Authorization
   - Seller approval
   - Buyer signature
   - Transfer eligibility


