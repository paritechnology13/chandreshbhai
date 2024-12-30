#!/bin/bash

# Define variables
LOG_FILE="/appserver/svoltp/batchloadgovt/camctr/logs/master_script.log"
ARCHIVE_DIR="/appserver/svoltp/batchloadgovt/camctr/archive"
TEMP_DIR="/appserver/svoltp/batchloadgovt/camctr/parser/temp"

# Function to log messages
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$LOG_FILE"
}

# Function to handle errors
check_error() {
    if [ $? -ne 0 ]; then
        log "Error occurred in the previous step. Exiting."
        exit 1
    fi
}

log "Starting the batch process"

# Step 1: Check for pending files
log "Executing Step 1: Check for pending files"
/appserver/svoltp/batchloadgovt/camctr/parser/oldupchk.sh
check_error

# Step 2: Decrypt the file and validate
log "Executing Step 2: Decrypt and validate file"
/appserver/svoltp/batchloadgovt/camctr/parser/olstgpre.sh
check_error

# Step 3: Load file into staging table
log "Executing Step 3: Load file into staging table"
/appserver/svoltp/batchloadgovt/camctr/stgloader/olstgldr.sh
check_error

# Step 4: Validate SSN and duplicates
log "Executing Step 4: SSN validation and duplicate check"
/appserver/svoltp/batchloadgovt/camctr/parser/olssndupchk.sh
check_error

# Step 5: Convert to batch loader format
log "Executing Step 5: Convert to batch loader format"
/appserver/svoltp/batchloadgovt/camctr/parser/olcareturn.sh
check_error

# Step 6: File transmission
log "Executing Step 6: Transmitting files"
/appserver/svoltp/batchloadgovt/camctr/reports/ndmcaret.sh
check_error

# Step 7: Post-process fraud records
log "Executing Step 7: Post-process fraud records"
/appserver/svoltp/batchloadgovt/camctr/processor/olbldca_fraudhold.sh
check_error

# Step 8: Update CardProd batchlog ID
log "Executing Step 8: Update CardProd batchlog ID"
/appserver/svoltp/batchloadgovt/camctr/processor/olbldca_update.sh
check_error

# Step 9: Consolidate counts and validate
log "Executing Step 9: Consolidate counts and validate"
/appserver/svoltp/batchloadgovt/camctr/parser/olchkcnt.sh
check_error

# Step 10: Generate return file
log "Executing Step 10: Generate return file"
/appserver/svoltp/batchloadgovt/camctr/parser/olcareturn.sh
check_error

log "Batch process completed successfully"
